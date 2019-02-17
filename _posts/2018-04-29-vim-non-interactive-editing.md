---
layout: post
title: Non-Interactive Text Editing in Vim
image: /assets/vim-efm/qfwindow.png
---

The Vim text editor has its roots in the traditional line-based editors of the early days. Due to that heritage, it supports a set of impressively powerful `Ex` commands that act on individual lines. In this article, we will see how to create a script composed of these commands and execute it from the shell.

## Ex mode
If you would like to see the 'original' `ex` editor in action, you can call vim with the `-e` option:
```
vim -e <demo.txt>
```

In this mode, you can't even see the actual file content yet -- probably that's why it's called *line-based*. To print all lines, enter `%print` at the Ex prompt. The `%` character means "all lines". In contrast, `1print` shows just the first line of the file.

But it's not only the traditional ex commands that are supported; you can also apply Vim normal mode commands to a range of lines. For example, `%normal Afoo` would append the word 'foo' to *all* lines of the file.

**This is great news**: We can solve an editing problem in interactive Vim, store the commands and apply it to thousands of instances of the same problem afterwards.

## Executing Ex scripts

Let's see how we can do this step-by-step. First, we will create a file `demo.txt`:
```
foo bar
bar foo
```

As an example, we're going to convert the words in the second column to uppercase so the converted file would look like:
```
foo BAR
bar FOO
```

In interactive Vim, we might end up applying the sequence `wgUW` to both of the lines in the file, starting from the leftmost column. This would scale perfectly to *N* lines, so let's put it into a script and call it `commands.vim`:
```
%normal wgUW
write!
quit
```

After that, we start Vim in *silent Ex mode* and pass the script contents via stdin:
```
cat commands.vim | vim -es demo.txt
```

We may also loop through a list of files:
```
for f in *.txt; do
	cat commands.vim | vim -es "$f"
done
```

We could even place the commands right into a bash script, using a "Here document", if that's desirable:
```
vim -es demo.txt <<EOF
	%normal wgUW
	write!
	quit
EOF
```

## Conclusion, References

Scripting Vim might be a good alternative to using *sed* and *awk*, provided that you are familiar with that editor anyway.

Below you will find some resources worth reading.

- [Learn Vimscript the Hard Way](http://learnvimscriptthehardway.stevelosh.com/) - Mostly about customizing Vim; chapter 29 is particularly interesting.
- [Practical Vim](https://pragprog.com/book/dnvim2/practical-vim-second-edition) - Great explanations on Ex mode (chapter 5) and a lot more.
- `:help mode-switching` - Good overview of available Vim modes
- [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/here-docs.html) - Chapter 19: Here Documents
