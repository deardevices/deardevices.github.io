---
layout: post
image: /assets/wrongabstr.png
---

Here are some thoughts on refactoring, especially on how to deal with duplicate code.

We all know that duplicate code is bad for a number of reasons[^1]. To eliminate it, we might extract a function, for example. In theory, that's fine. A recent [twitter post](https://twitter.com/Domysee/status/1057315981150875648) made me aware of a pattern regarding function extraction I've noticed over the last few years:

> The wrong abstraction is worse than duplication.

Apparently, this whole idea comes from [Sandi Metz](https://twitter.com/sandimetz), as she wrote on [her blog](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction) in 2016. Anyways, this describes precisely what I observed over time. There is some *extra thinking* needed in transforming several more or less redundant code blocks into one single function.

## Deducing Underlying Concepts

Before extracting a function, we might ask questions like:

- What concept or idea does the code share?
- How to name the new function?
- What would its arguments be?

After all, the original code usually doesn't have a name or concept assigned to it; those have to be *deduced from context*. If we get it wrong at this point, our refactoring still might be successful in a way that duplication is removed. The problem is though, that we have introduced some common concept that *isn't really there*. This may point readers of our code (including our future selves) in the wrong direction.

Let's take a look at an example. The following two code blocks from a microcontroller program are almost identical:

```c
// blink Led
Gpio gpio;
gpio.setPin(1, true);
sleep(1);
gpio.setPin(1, false);
```

```c
Gpio gpio;
gpio.setPin(2, true);
sleep(1);
gpio.setPin(2, false);
```

Both pull up an output pin for one second. The difference is just in the *pin number*. To resolve this duplication, let's extract a function using the first block as a template:

```c
void blinkLed(Gpio& gpio, int index) {
  gpio.setPin(index, true);
  sleep(1);
  gpio.setPin(index, false);
}
```

This refactoring seems reasonable at first sight. But who said both original code snippets were about driving LEDs? For the first one, there's a comment suggesting that, but the second one might as well turn on a servo motor for one second.

That way, we have introduced a concept of an LED blinking that doesn't really help the reader of the program. It doesn't cover the second code block so it will lead to misunderstanding.

Note: The same can be shown for extracting entire classes, although there's even more that can go wrong - the class itself has to be given a name, ideally a bit more concrete than Controller, Manager, etc.

## Conclusion

We've seen an (admittedly trivial) example of a well-meant refactoring that, instead of improving code quality, made the situation even worse. We need to make ourselves aware of the energy we have to put into refactorings we make. In practice, I tend to follow a rule to avoid doing any harm:

> If in doubt, better leave the code as it is -- or ask someone on the team who might be more familiar with that part of the project.

## References
- [Duplicate Code (Wikipedia)](https://en.wikipedia.org/wiki/Duplicate_code)
- [Refactoring (Martin Fowler)](https://martinfowler.com/books/refactoring.html)
- [The Wrong Abstraction (Sand Metz)](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction)

[^1]: Refer to Martian Fowler's [book on Refactoring](https://martinfowler.com/books/refactoring.html) to find out why.
