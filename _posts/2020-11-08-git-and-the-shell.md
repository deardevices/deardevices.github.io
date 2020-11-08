---
layout: post
title: "Git diff colliding with the shell: special characters"
image: "/img/git.png"
---

It sounds simple enough: You would like git to show local changes, but considering certain file types only. Something like `git diff *.h` usually does the job but in some cases yields unexpected results. We're going to find out what this is about in this article.

The following directory listing shows the state of a (pretty small) git repo I created as an example:
```
$ find . | grep -v ".git"
.
./bar.h
./foo
./foo/foo.html
./foo/foo.c
./foo/foo.h
```

It contains a few c, h, and html files. To show which of the files have local changes we can use `git diff`:
```
$ git diff --name-only
bar.h
foo/foo.c
foo/foo.h
foo/foo.html
```

This quickly reveals that all of the files have local modifications (just because I've arranged them this way). By checking with the actual changes I did I can confirm that this is the complete list.

Occasionally I find it useful to limit the output of `git diff` to files of a certain type. Filtering for header files only yields a surprising result though:
```
$ git diff --name-only *.h
bar.h
```

From git's output, it looks like only one of the two header files had changed -- but in fact it should be two of them, as you can see from the listing above.

Apparently, the file from the `foo` directory is missing in the output.


If we do the same for c files, the result looks alright:
```
$ git diff --name-only *.c
foo/foo.c
```

This feels weird: why would git list changes from the `foo` directory only for certain file types?

## It's interfering with the shell!
After some searching through the web, I found a great pointer on Stack Overflow: [git diff for cpp files prints nothing in one repository](https://stackoverflow.com/questions/54124269/git-diff-for-cpp-files-prints-nothing-in-one-repository).

It turns out that the strange behavior is because the asterisk sign `*` is a special character for the shell we use to type in our git commands. The reason why the call to git is behaving differently for c and h files is simply because there's an h file inside the working directory!

Let's look at a simplified example, replacing the call to git by echo. That way we can focus on how the shell expands the asterisk sign:
```
$ echo *.h
bar.h

$ echo *.c
*.c
```

Here's the difference: Because there's a header file where we execute the shell, the expression `*.h` gets evaluated to `bar.h` while `*.c` remains unchanged (i.e. the shell couldn't resolve it).

Therefore, for header files, what eventually gets passed to the git command looks like this:
```
$ git diff --name-only bar.h
```

This shows that git works as expected; the issue lies within the special meaning of `*` for the evaluating shell.

## How to fix it
There are certain ways to fix this, as mentioned in the Stack Overflow article linked above:
1. Quote the `*` character: `git diff \*.h`
2. Put the entire argument in quotes (or double quotes): `git diff "*.h"
3. Explicitly tell git that the argument is a path: `git diff -- *.h`

While the first two are 'fixing' the behavior of the shell, the third one is a feature of git: According to [`git help diff`](https://git-scm.com/docs/git-diff), you can use `--` to explicitly state that everything that follows is to be interpreted as a path.

## Conclusion
This weird behavior has bugged me for a while, mainly because I had a hard time coming up with a clear description of the issue. Now I'm happy there's a good explanation and I hope it helps you as well.

(Git logo from [git-scm.com/downloads/logos](https://git-scm.com/downloads/logos), created by [Jason Long](https://twitter.com/jasonlong).)
