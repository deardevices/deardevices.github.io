---
layout: post
---

When working on the command line, you may sometimes find yourself in a situation where you'd like to start a new program while another long-running process (e.g. an editor) is **blocking the current terminal**. In the course of this article we are going to find out how to achieve quick context switching in such a case, without the need for opening up an entirely fresh terminal window.

## Basic usage

Bash provides a set of commands that deal with the topic of **job control**. You can use them together with the suspend character `Ctrl-Z` [^1]. Let's take a look at the following state diagram that shows the lifetime of a job[^2]:

![]({{ "/assets/job-control/graph1.dot.png" | absolute_url }})

Starting a new process brings it into *foreground*. In order to start another program at this point, you can hit `Ctrl-Z` to *suspend* the currently running process. Note that in this state, the process is not functional anymore. To resume it, type `bg` or `fg` to continue execution in background or foreground, respectively.
While the process is executed in *background*, it does not respond to any keyboard-generated signals anymore. That is, you need to bring it into foreground to be able to kill it by hitting `Ctrl-C`.

## Job overview
To see which jobs are running in the current session, you can use the `jobs` bash builtin. The following listing shows its output after starting and suspending three instances of the vim editor.

```
[me@mymachine ~] [3]$ jobs
[1]   Stopped              vim foo
[2]-  Stopped              vim bar
[3]+  Stopped              vim baz
```

Note that the current and previous jobs are marked with `+` and `-` respectively.

Did you notice the `[3]` in the prompt of the listing above? This is a nice little extension to the `PS` environment variable that shows the **number of currently managed jobs**. That way you're always aware of the existence of any background job without calling `jobs` over and over again:

```
export PS1='\u@\h \W[\j]\$ '
```

## References
- [Advanced Bash Scripting Guide - Job Control Commands](http://tldp.org/LDP/abs/html/x9644.html)
- [Bash Reference Manual: Job Control Basics](https://www.gnu.org/software/bash/manual/html_node/Job-Control-Basics.html)

[^1]: The *suspend* character is interpreted by the terminal (kernel driver), not bash; refer to the output of `stty -a`.
[^2]: Actually, a job in Bash may consist of several processes started via a pipeline.

