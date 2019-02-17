---
layout: post
image: /assets/readline.png
---

When developing command line applications, you will most certainly want to have bash-like line-editing features. For that, an excellent choice is the GNU readline library. We will see in this article how to use its features from out own programs in order to provide a user experience similar to that of many free software projects out there.

## What your users expect
Even without mentioning it explicitly, everyone prefers *similar* applications to have *similar* behavior.

When it comes to command line apps, that often boils down to command-line editing features. Look at some prominent examples -- the bash shell, the GNU debugger and the interactive Python interpreter: They all offer the exact same line-editing capabilities.

They all support:
- Stepping back and forth in history.  
- Reverse history search (`<Ctrl-R>`).
- `<TAB>` completion for their own command set.

And, most notably they all respect the user's presets from `~/.inputrc`, i.e.
- emacs/vim mode
- custom key mappings

How do these programs manage to offer the exact same line-editing behavior? If we look at their dependencies, we will find out that  they rely on the [GNU readline](https://en.wikipedia.org/wiki/GNU_Readline) library for that task.

## How to do it in C
To find out more about the GNU readline library, let's start at the corresponding man page (`man 3 readline`). Actually, this document is a very good read -- it explains everything you need to know in just about 800 lines.

Let's start with an example.

For simple line editing, it's already sufficient to use the `readline()` function instead of `gets()` or `scanf()` where user input is read.

The program in the following snippet shows a prompt to the user and echoes user input.

```c
/* rl_example0.c */

#include <stdlib.h>
#include <readline/readline.h>

int main() {
  char* input = readline("> ");
  if (input) {
    printf("%s\n", input);
    free(input);
  }
  return 0;
}
```

Note that although this simple approach lacks history features, it offers powerful line-editing already, like `<Ctrl-w>` for killing a word and `<Ctrl-a>` for moving to the beginning of the line.

If you set `editing-mode` in `~/inputrc` to `vi`, you can even use some vi keys to navigate on and edit the current line.

To make history work, we additionally call `add_history()` and put everything into an endless loop:

```c
/* rl_example1.c */

#include <stdlib.h>
#include <readline/readline.h>
#include <readline/history.h>

int main() {
  while(1) {
    char* input = readline("> ");
    if (!input)
      break;
    if (*input)
      add_history(input);
    printf("%s\n", input);
    free(input);
  }
  return 0;
}
```

This enables navigating through the history using the `Up`/`Down` arrow keys as well as reverse history search via `<Ctrl-r>`.

## Conclusion
The GNU readline library is a good choice to get powerful line editing features. In an upcoming article we will dive into another very important feature of readline we haven't covered yet: `<TAB>` completion.

Please refer to the [readline-intro](https://github.com/ronalterde/readline-intro.git) repository on github for a copy of the example above, including a Makefile.
