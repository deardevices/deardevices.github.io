---
layout: post
---
In Linux, you can execute an interpreted script file in exactly the same way as you would execute a compiled binary: by setting the 'executable' attribute and just run it. A special sequence of characters at the beginning of the script makes this possible. In this article we will explore how this -- pretty elegant -- way of script execution works and which components of the Linux system are involved.

Consider the following Python script. It's introduced by two special characters and also contains the path to the python interpreter.

```python
#!/usr/bin/python3
# file: demo.py
print("Hello World!")
```

To the interpreter, every line beginning with a *#* is treated as a comment. We might use it as a hint and run the script by calling the interpreter directly:

```sh
$ /usr/bin/python3 demo.py
Hello World!
```

To actually utilize the two introductory characters -- the [*shebang*](https://en.wikipedia.org/wiki/Shebang_(Unix)) sequence, as it's called -- let's run the script like that:

```sh
$ chmod u+x demo.py
$ ./demo.py
Hello World!
```

I'm sure you're aware of that already. But don't you think this behavior is rather **surprising**? How can we make a computer execute a script that **doesn't even contain any instructions** the machine might understand?

## Who handles the shebang?

There must be a place in the system where the interpreter is called with the script path as an argument.
- Does the shell parse the first line of the script?
- Does the kernel know about scripts?

Let's try to figure out how "real" programs are usually executed. As described by Robert Love in his book [*Linux System Programming*](http://shop.oreilly.com/product/9780596009588.do), running a program in Linux is performed within two steps: first a new process is created by *forking* the currently running one, then a new program is loaded into that process by *executing* it. We're going to focus on the second step -- loading. It is made available to the programmer through a family of *exec()* functions that ultimately result in a call to the *execve* system call. Let's take a look at the [man page](https://linux.die.net/man/2/execve) of the corresponding library function which gives an important hint:
> An interpreter script is a text file that has execute permission enabled and whose first line is of the form:
>    #! interpreter [optional-arg]

As the *execve()* library function is just a thin wrapper around the Linux system call, it seems to be the kernel's responsibility to do the actual execution of the script.

## Binary Formats
It turns out an *interpreter script*, as mentioned in the man page, is just another "binary" format known to the kernel. It's contained with the module [*binfmt_script*](https://elixir.bootlin.com/linux/v4.15.6/source/fs/binfmt_script.c) we can find in the *fs* directory of the source tree. In its init function, this module registers a function *load_script()* that's called upon execution of an interpreter script:

```c
/* fs/binfmt_script.c */
static int load_script(struct linux_binprm *bprm)
{
	[...]

	if ((bprm->buf[0] != '#') || (bprm->buf[1] != '!'))
		return -ENOEXEC;

	[...]
}
```

**There it is**: the check for the two characters that make up the shebang sequence: *#!*

After finding out the interpreter's path, *load_script()* eventually makes a call to [search_binary_handler()](https://elixir.bootlin.com/linux/v4.15.6/source/fs/exec.c#L1616) which in turn will try to load the program:

```c
int search_binary_handler(struct linux_binprm *bprm)
{
	[...]
	/* This allows 4 levels of binfmt rewrites before failing hard. */
	if (bprm->recursion_depth > 5)
		return -ELOOP;
	[...]
	list_for_each_entry(fmt, &formats, lh) {
		[...]
		retval = fmt->load_binary(bprm);
		[...]
	}
[...]
}
```

Note that there is a recursion limit in the code. This makes sense if we take into account that *load_script()* is a **polymorphic overload** of *load_binary()* -- this is accomplished by registering a *struct linux_binfmt* instance with the system.

We can trigger that limit by creating a file */tmp/demo* and insert its own path as the interpreter after the shebang:

```sh
#!/tmp/demo
```

Executing that script will result in the error message for the error code *ELOOP*:
```sh
$ chmod a+x /tmp/demo
$ /tmp/demo
-bash: /tmp/demo: /tmp/demo: bad interpreter: Too many levels of symbolic links
```
