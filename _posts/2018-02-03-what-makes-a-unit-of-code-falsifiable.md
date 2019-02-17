---
layout: post
---

In his recent book about Software Architecture, Robert C. Martin a.k.a. Uncle Bob describes programming as a discipline of science, rather than mathematics. In one of the first chapters of the book he talks about Structured Programming and how it enables us to produce -- according to the scientific method -- falsifiable, i.e. testable units of code.

I really like the idea to think about programming that way. There is one thing I do not quite understand, though. Why is Structured Programming guaranteed to produce testable units? (Disclaimer: I haven't found an answer yet.)

## Proving wrong code wrong
Before trying to find an answer to this question, let's approach the problem from another direction: by creating a falsifiable unit using a building block defined by Structured Programming. According to Dijkstra's original publication from 1986, [Go To Statement Considered Harmful](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf), we may choose from sequence, selection, repetition and procedure. So let's create a function that's simply wrong:
```c
/* file add.c */

unsigned int add(unsigned int a, unsigned int b) {
    if (a == 5)
        return a;
    else
        return a + b;
}
```

```c
/* file add.h */

unsigned int add(unsigned int a, unsigned int b);
```

The following test is enough to prove the *add* function wrong, without even considering the implementation:

```c
/* file test_add.c */

#include "add.h"

TEST(TestAdd, worksForSomeInputs) {
    ASSERT_EQ(8, add(5, 3));
}
```

As I understand, if we are able to falsify a particular statement (or function), this also means that statement is falsifiable. This little experiment doesn't tell us anything about the opposite, though. How do we create something that is *not* falsifiable?

## Programmer-independent coordinate system
To understand the issue related to the goto statement, let's see what Dijkstra tells us about it:
> The unbridled use of the go to statement has an immediate consequence that it becomes terribly hard to find a meaningful set of coordinates in which to describe the process progress.

By *process*, a running instance of a program is meant:

> [...] to make the correspondence between the program (spread out in text space) and the process (spread out in time) as trivial as possible.

As furthermore stated, to track the progress of such a process a particular *set of coordinates* is needed. If, for example, all we have is just a sequence of statements, the only coordinate necessary is a pointer into the program text. Consider the following program:
```c
a = 5;
b = 6;
c = 7;
a = b;
a = c;
```

To tell the state of the process this program is executed in, i.e. the value of the variables a, b and c, all we need to know is the indeindex of the line to be executed in the next processor cycle. Dijkstra calls this a "textual index". The following table shows how such an interpretation might look like, assuming all the variables are zero-initialized.

| line index | a | b | c |
|-------|---|---|---|
| 1     | 0 | 0 | 0 |
| 2     | 5 | 0 | 0 |
| 3     | 5 | 6 | 0 |
| 4     | 5 | 6 | 7 |
| 5     | 6 | 6 | 7 |
| 6     | 7 | 6 | 7 |

One can follow along the state of the process pretty easily by looking at the table. The correspondence to the program code is always obvious.

## Attempt to construct something non-testable
It feels like if we manage to create such an example incorporating "unrestrained" use of goto, we may actually end up with some untestable unit. How do we know if we're there yet? Non-falsifiable units are supposed to be *not testable*, after all.

Uncle Bob states that an "unrestrained use of goto" leads to "unprovable" units of code. What does it mean to use the goto statement in such a way?
