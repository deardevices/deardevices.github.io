---
layout: post
title: "Raspberry Pi: How to cross-compile against third-party libraries"
---

Cross-Compiling for Raspberry Pi can become pretty challenging as soon as third-party libraries get involved. Using the *sysroot* concept together with *rsync*, it's easy to keep those libraries synchronized between your host and target system.

Note: This is a follow-up to [How to Cross-Compile for Raspberry Pi on Ubuntu Linux in 5 Steps]({{ site.baseurl }}{% post_url 2019-04-18-how-to-crosscompile-raspi %}). Some parts are taken from the [Qt Wiki](https://wiki.qt.io/RaspberryPi2EGLFS).

In this post, we will attempt to cross-compile a self-contained example (for Raspbian Stretch) that relies on some third-party library. You will see how to get development packages ready on your Pi and how to synchronize them to your host computer.

## 1 Install development libraries on the Pi
Before starting cross-compilation, we want to make sure we've got all the required libraries at hand.

To achieve that, we will first install them on the target machine (the Pi). Usually, there's a `*-dev` package for every library, containing headers and object files.

Let's use [ncurses](https://en.wikipedia.org/wiki/Ncurses), a library for creating text-based user interfaces, as an example. SSH into your Pi and install the corresponding package:
```
sudo apt-get install libncurses5-dev
```

At this point, compiling against libncurses would probably already work locally on the Pi, assuming a working toolchain. Our cross-compiler, however, isn't yet aware of that change.

## 2 Synchronize the Pi's directories to the host machine
For making cross-compilation work, we may copy those development libraries over to the host system so we can compile & link against them. 

We can let `rsync` do the job. That way, it's easy to keep everything in sync in the future as well.

Assuming you already got a working directory on your host under `/opt/pi`, as introduced in [How to Cross-Compile for Raspberry Pi]({{ site.baseurl }}{% post_url 2019-04-18-how-to-crosscompile-raspi %}), create a *sysroot* directory and fetch all relevant content from the Pi:
```
cd /opt/pi
mkdir sysroot
cd sysroot

rsync -avz pi@pi.local:/lib .
rsync -avz pi@pi.local:/usr/include usr
rsync -avz pi@pi.local:/usr/lib usr
```
Note: Replace `pi.local` by your Pi's host name or IP address.

You should now have a copy of the relevant headers and libraries:
```
$ find -name libncurses.so.*
./lib/arm-linux-gnueabihf/libncurses.so.5.9
./lib/arm-linux-gnueabihf/libncurses.so.5

$ find -name ncurses.h
./usr/include/ncurses.h
```

The output of the `find` command might vary depending on your Raspbian version. Just make sure you got the header file and the shared object (note that one of the `*.so` files is just a symlink).

## 3 Attempt to Cross-Compile without setting *sysroot*

You're now ready for cross-compiling.

Here's an example from the [NCURSES Programming Howto](`http://www.tldp.org/HOWTO/NCURSES-Programming-HOWTO/helloworld.html). Save the following snippet as `demo.c`.
```c
#include <ncurses.h>

int main()
{	
  initscr();
  printw("Hello World !!!");
  refresh();
  getch();
  endwin();

  return 0;
}
```

Let's determine which cross-toolchain to use by setting the `PATH` environment variable accordingly:
```
export PATH="$PATH:/opt/pi/tools/arm-bcm2708/\
gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin"
```

Now you should able to launch the cross-compiler. Make sure you can start it like this:
```
$ arm-linux-gnueabihf-gcc --version
arm-linux-gnueabihf-gcc (crosstool-NG linaro-1.13.1+bzr2650 - Linaro GCC 2014.03) 4.8.3 20140303 (prerelease)
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Let's just try to build without further considerations:
```
$ arm-linux-gnueabihf-gcc demo.c -o demo -lncurses
demo.c:1:21: fatal error: ncurses.h: No such file or directory
 #include <ncurses.h>
                     ^
compilation terminated.
```

This is failing as expected: the compiler is missing a certain header file.

## 4 Cross-Compile with setting *sysroot*

We can fix the build by adding the `--sysroot` option when calling gcc. This tells the compiler to use the specified location as the *logical root directory for headers and libraries*, as stated in the [GCC docs](https://gcc.gnu.org/onlinedocs/gcc/Directory-Options.html).

Call the compiler like this:
```
arm-linux-gnueabihf-gcc --sysroot ./sysroot demo.c -o demo -lncurses
```

And verify the executable has been created successfully.
```
$ file demo
demo: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, BuildID[sha1]=0f88dffd576bab754342516b3566868fd64f07a1, not stripped
```

The `demo` program is now ready to be executed on your Pi:
```
scp demo pi@pi.local:~/
ssh pi@pi.local
pi@pi $ ./demo
```

## References
- [RaspberryPi2EGLFS (Qt Wiki)](https://wiki.qt.io/RaspberryPi2EGLFS)
- [GCC docs: Options for Directory Search](https://gcc.gnu.org/onlinedocs/gcc/Directory-Options.html)
- [Example from NCURSES Programming Howto](`http://www.tldp.org/HOWTO/NCURSES-Programming-HOWTO/helloworld.html)

