---
layout: post
title: Stand-alone TDD monitor using 'entr'
image: /assets/tdd_monitor.png
---

Have you ever thought of something that continuously monitors your project's source tree and triggers a unit test build upon file changes? You want it to be independent of the editor or IDE you use? Then maybe this is for you.

Actually, this idea came to my mind some time ago: Just open a dedicated terminal window and call `make` at a regular interval, let's say every 30 seconds. To visualize the build result, you may set the terminal background color to either 'red' or 'green' as soon as the process finishes.

```bash
while true; do
  make all
  set_terminal_color $?
  sleep 30
done
```

Note that `set_terminal_color` is meant as a placeholder for a shell function responsible for coloring the terminal (window).

This simple approach is fine in a way that it is absolutely *independent of your current editor or IDE*. There is one serious drawback though that makes it rather hard to use in practice: the build is totally **out of sync to your editing workflow**. In worst case, you would need to wait for an entire cycle to see the effect of a single change you just made.

That's why I gave up on the idea at that point. Last week, however, I spotted a [Twitter post by @schmonz](https://twitter.com/schmonz/status/1049489149080821760) that made me aware of a tool called 'entr'. Its man page title reads:
> run arbitrary commands when files change

## Change-Driven Approach

This sounds like the missing piece of the previous concept. If we only started the unit test build upon changes in the sources, we would only need to wait for the actual build to finish. There would be no fixed (and arbitrary, to be honest) monitoring cycle.

Here's how the `entr` tool is called:
```bash
ls | entr -s 'make all; set_terminal_color $?'
```

The program expects a list of file names at its standard input. As an example, we simply pass the output of `ls` to tell it to watch all files in the current directory.

For reference, here's an idea for an implementation of `set_terminal_color`:
```bash
# Put this into 'functions.sh' and use it like this:
# ls | entr -s '. functions.sh; make; set_terminal_color $?'
set_terminal_color () {
  if [ $1 -eq 0 ]; then
    setterm -term linux -back green;
  else
    setterm -term linux -back red;
  fi
}

```

## Conclusion

The example introduced in this article is pretty basic. But what it shows is a robust approach to check the current status of your TDD build from a single terminal window.

There's definitely more we can do to improve it, like:
- Incorporating unit test execution, not just `make`.
- Better (and prettier) terminal coloring.

**What do you think?** Actually, I'm not sure whether we should really separate the TDD monitoring from the IDE in such a way. Maybe a tighter integration would be better?

## References
- [Entr Project]( http://entrproject.org/ )
- [Advanced Bash-Scripting Guide: Functions](http://tldp.org/LDP/abs/html/functions.html)
