---
layout: post
title: "Cross-compiling for Raspberry Pi: QEMU User Emulation and Dynamic Linking"
---

This is a follow-up to an [article]({% post_url 2019-04-18-how-to-crosscompile-raspi %}) I wrote back in 2019 that was about linking *statically* against libc. This time we are going to take a look at an alternative way: dynamic linking.

The previous article presented a way to cross-compile a Hello World program for the Raspberry Pi and run it on the host machine using QEMU User Emulation.

To keep it simple, we focused on linking statically against the C Standard Library (libc). In particular, I mentioned concerns about how it would be more complex to set up dynamic linking instead. Although maybe not obvious enough, my comment at the end was directed towards the `-static` option passed to the compiler in a snippet above. I should have been more precise when I wrote:
> The principle is the same for dynamic linking, it’s just a bit more complex to set up (because you have to deal with selecting the loader and such).

Also, note that this is not an issue for running on the target system, as the loader will be available at the expected location under `/lib`.

As a side note: there is yet another article from Dec-2019 ([How to cross-compile against third-party libraries]({% post_url 2019-12-25-raspberry-pi-sysroot %})) that talks about linking against a third-party library and running on the target system (without qemu-user). This is fine - but no emulation involved there.

But now, let's take a closer look at how to link a program dynamically against libc and run it on the Host via QEMU.

## Where we left off

Part of the previous article was setting up the toolchain. Assuming you have gone through these steps already, running the following two commands should get you a compiled binary called `hello_static` (see below for the contents of `hello.c`):

```
export PATH="$PATH:/opt/pi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin"
arm-linux-gnueabihf-gcc -static hello.c -o hello_static
```

```c
/* hello.c */
#include <stdio.h>
int main(void)
{
  printf("Hello Cross Compiler!\n");
  return 0;
}
```

To verify, run the `file` command on the resulting binary. It confirms you've created an *ARM* executable that is statically linked:
```
$ file hello_static 
hello_static: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, for GNU/Linux 2.6.26, BuildID[sha1]=889bdaafd9297d9da02572bca1af09d238e5f6d4, not stripped
```

Previously, we mentioned a limitation for practical use with this. To be more specific: the size of the resulting executable is dramatically different between the two.

## Now: compiling without the 'static' option

Just as before, we can compile `hello.c` for the target - but leave out the `-static` option:
```bash
arm-linux-gnueabihf-gcc hello.c -o hello_dyn
```

Consider the output below. The statically linked executable is 400 (!) times larger than the one that was linked dynamically.

```bash

$ size hello_static
   text    data     bss     dec     hex filename
 480852    1996    6400  489248   77720 hello_static

$ size hello_dyn
   text    data     bss     dec     hex filename
    912     284       4    1200     4b0 hello_dyn
```

This rather dramatic difference in size may vary, depending on how the library (libc in this case) is structured.

Are you interested in what exactly is contained with static one, making it so large?

Go ahead and use `objdump` to display the machine instructions and symbols:
```
arm-linux-gnueabihf-objdump --disassemble-all hello_static | less
```

When looking at the output of this command, you will notice symbols that look familiar, e.g. `strcpy`, `memset`, and `fprintf`.

So, regardless of the reason we decide to link dynamically, what is the complication with that?

## The issue with running a dynamically linked executable
As with the statically linked one, let's try to run the dynamically linked executable on the *Host* system. Note that this utilizes QEMU, as discussed before.
```
$ ./hello_dyn 
/lib/ld-linux-armhf.so.3: No such file or directory
```

It fails. What does this error message tell us?

Taking a closer look at the output of `file hello_dyn`, there is an observation to make. Can you spot the hint?

```
hello_dyn: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV),
dynamically linked, interpreter /lib/ld-linux-armhf.so.3,
for GNU/Linux 2.6.26, BuildID[sha1]=affef4651eb9f30c43e7447abf71e27115bc8952,
not stripped
```

Apparently, there is something called an *interpreter* that is missing. We may want to query more details by running the `readelf` utility:
```
arm-linux-gnueabihf-readelf --program-headers hello_dyn
```

Among other things, this shows:

```
[Requesting program interpreter: /lib/ld-linux-armhf.so.3]
```

Well, that still doesn't help much for explaining what an interpreter is and where to find it. Presumably, the man page of ld-linux can shed some light on this. Run `man ld-linux` to show them. Note that content in there might be specific to the *Host* system - as the target is a Linux system as well, I don't see much concern. I think it is fair to assume it represents well enough what's going on on the target as well.

These are the things to highlight here:
1. The missing interpreter is the *dynamic linker*.
1. It is responsible for finding and loading shared libraries needed by a program.
1. It also takes care of running the program after that.

That's the answer to the question we asked before: the dynamic linker is the missing piece.

## Where's the dynamic linker located?
The name `ld-linux-armhf.so.3` suggests that we're dealing with a program that is supposed to run on the target system. I can confirm it is available on the Raspberry Pi's file system, which explains why the program *just runs* when we copy it over to the Pi.

What we are looking for is a way to point the QEMU emulation to where that linker is located on the *Host* file system.

Well, it turns out that it is part of the sysroot directory:
```
$ cd /opt/pi/tools/
$ find -name ld-linux-armhf.so.3
./arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/arm-linux-gnueabihf/libc/lib/arm-linux-gnueabihf/ld-linux-armhf.so.3
./arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/arm-linux-gnueabihf/libc/lib/ld-linux-armhf.so.3
./arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/arm-linux-gnueabihf/libc/lib/arm-linux-gnueabihf/ld-linux-armhf.so.3
./arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/arm-linux-gnueabihf/libc/lib/ld-linux-armhf.so.3
./arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/ld-linux-armhf.so.3
```

The man page `man qemu-arm-static` tell us how to make qemu aware of its location:

```
-L <path>
  Set the elf interpreter prefix (default=/etc/qemu-binfmt/%M).
```

That means we *can* run the dynamically linked program on the host: by calling qemu explicitly, like this:
```
$ export INTERPRETER=/opt/pi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot
$ qemu-arm-static -L $INTERPRETER hello_dyn
Hello Cross Compiler!
```

## Summary, References
We described a way to run a cross-compiled, dynamically linked Hello World program on the Host machine. See below for a list of references that might be useful.

- [qemu-user-static man page](https://manpages.debian.org/testing/qemu-user-static/qemu-arm-static.1.en.html)
- [ld-linux man page](https://linux.die.net/man/8/ld-linux)
- [Debian Wiki: QemuUserEmulation](https://wiki.debian.org/QemuUserEmulation)
- [Arch Wiki: QEMU](https://wiki.archlinux.org/title/QEMU)
- [QEMU docs](https://qemu-project.gitlab.io/qemu/user/index.html)
