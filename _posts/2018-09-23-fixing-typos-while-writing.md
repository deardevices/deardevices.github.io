---
layout: post
title: Fixing Typos while Writing -- Two Ideas
---

Here are some thoughts on text editing. No matter what kind of text you write -- source code, articles, to-do lists, books -- there's always a chance you get a single letter wrong at one point. Depending on your touch typing skills, this might not happen very often. But *when* it happens, **what do you do?**

- Do you press and hold the backspace key until you get back to the typo?
- Do you jump back to the bad letter, fix it like a surgeon[^1] and continue writing?

## My experience
Personally, I used to (rather unconsciously) adhere to the second approach. One day, however, I found out that even the Vim editor has support for [deleting the last word before the cursor](http://vimhelp.appspot.com/insert.txt.html) while typing in *insert* mode. This is not typical though. Usually, there are two steps involved: first, you use movement commands to get to the desired text position and then enter *insert* mode.

Although I'm a big fan of Vim's fantastic movement commands, this made me change my view to another perspective. What kind of habit would you develop if every typo was really **easy to fix?** Your brain may think something like: "It's not critical to get it right the first time -- it's not a big deal anyway". So there's **no motivation** for typing words correctly on the first attempt.

## Better use the first approach?
To me, deleting the entire last word sounds a bit drastic at first. You did put some effort into writing it in the first place, did you? Nevertheless, I think the training effect might be worth it. So I will try it for some time and see whether there's a positive effect on my typing.

What do you think? Are there any scientific studies about this? I would love to know your opinion and personal experience.

To be complete: the Vim editor is by far not the only one to implement the *delete last word* feature.
You can use `Ctrl-w` on the terminal and inside Emacs. Many GUI applications support `Ctrl-Backspace` for doing the same.

[^1]: Example for Vim: Let's say you wrote 'house' instead of 'horse', you can do '<ESC>Furr' to fix that.
