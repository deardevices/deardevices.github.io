---
layout: post
---

When writing conditionals in shell scripts, have you ever wondered whether that whitespace right after the opening bracket is really necessary for the expression to evaluate correctly? You might be surprised by the answer we'll discover in the course of this short article.

## Why isn't that working?
Consider the following snippet and its output when executed:
```bash
#!/bin/bash
# file: demo.sh

if ["a" == "a"]; then
  echo true
else
  echo false
fi
```

```
$ bash demo.sh
demo.sh: line 4: [a: command not found
true
```

Execution fails, but why? Is it really necessary to put whitespaces between the conditional expression and the enclosing brackets? Apparently, yes -- by fixing line 4, the script executes perfectly without emitting an error:

```bash
if [ "a" == "a" ]; then
```

```
$ bash demo.sh
true
```

What is the reason behind this? Actually, the initial error message gives a hint: the interpreter was looking for command *[a* -- which looks like a simple concatenation of what's in the text -- but couldn't find it. It seems like the name of an actual executable is expected after the *if* token.

Let's verify that by looking for such a command in the shell:
```
$ which [
/usr/bin/[

$ file /usr/bin[
/usr/bin/[: ELF 64-bit LSB  executable, x86-64, [...]
```

Seriously? There actually is a *left bracket* command available on our Linux system!

So let's make the original line a bit more explicit too see if that all fits together:
```bash
if /usr/bin/[ a == b]; then
```
```
$ bash demo.sh
true
```

It's still working!

## A quick look at the sources
Now that we know we're dealing with just an ordinary executable, let's examine its source code to see what's happening. On Debian-based systems we can find out which package a particular command belongs to and fetch its sources right away:
```
$ dpkg -S /usr/bin/[
coreutils: /usr/bin/[
$ apt-get source coreutils
```

This will download the corresponding source archive and extract it into a directory named *coreutils-8.21* or something similar. After some searching through the *src* directory, one might notice the file *lbracket.c*, which has only the following two lines of code:

```
$ cat src/lbracket.c
#define LBRACKET 1
#include "test.c"
```

This snippet suggests the *lbracket* command is just an alias for something else, the *test* command. This makes sense because the conditional expression in the original shell script could also be stated as:
```bash
if test "a" == "a"; then
```

So let's follow that pointer and look at the code for the test command (*test.c* in the same directory).

Right at the top of the file, we encounter another evidence for the alias hypothesis:
```c
#if LBRACKET
# define PROGRAM_NAME "["
#else
# define PROGRAM_NAME "test"
#endif
```

The only real difference to the *test* command seems to be: 
```c
  if (margc < 2 || !STREQ (margv[margc - 1], "]"))
    test_syntax_error (_("missing ']'"), NULL);
```

This is the check for the closing bracket -- which is only necessary for *[*, not for the *test* command which seems to be otherwise equivalent. Here you can see the effect of these statements:

```
$ /usr/bin/[
/usr/bin/[: missing ']'
```

## Conclusion
We've found out that everything that's inside the conditional statement in the shell is actually just a list of arguments to the '[' command. This is the reason why we the whitespace after the bracket is mandatory.

It's probably worth mentioning that -- for bash at least -- the *[* expression is also a shell builtin. That means it usually overrides the executable at */usr/bin*. Its behavior is almost identical though.

## References
- [original lbracket.c](http://git.savannah.gnu.org/gitweb/?p=coreutils.git;a=blob;f=src/lbracket.c;h=b57ca9bb0cfeedab045b42e2afa922516808797a;hb=HEAD)
- [original test.c](http://git.savannah.gnu.org/gitweb/?p=coreutils.git;a=blob;f=src/test.c;h=aae45012a46b77b2e2c9b4f8428acd457294c1be;hb=HEAD)
- [bash builtin for the test command](http://git.savannah.gnu.org/cgit/bash.git/tree/builtins/test.def?h=bash-4.4)
