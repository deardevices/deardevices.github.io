---
layout: post
title: "3 Practical Uses of void Pointers in the C Language (part 1/3)"
image: "/img/arrows.jpg"
---

Probably every beginner's book on C programming has a section on pointers. Usually, there are also one or two paragraphs on the topic of *void* pointers. Have you ever asked yourself what they can actually be used for? In this article, we will explore some **practical uses** for pointers of type void.

## Introduction

First of all -- what is a `void` pointer, anyway? A pointer of type `void` is, like every other pointer, a variable that contains an address into memory (i.e. RAM). The special property of void pointers is that they have *no assigned type* so the address is actually all they store, from a compiler's perspective.

You can take the address of a certain variable and use a void pointer to store it:
```c
uint16_t a = 0xabcd;
void * p = &a;
```

What you cannot do is *dereference* the pointer afterwards:
```c
*p = 0xfeed; // Fails
```

This makes sense: The compiler cannot infer what type of value is stored at the memory location referred by `p`. For instance, it doesn't know how big that hunk of memory may be. You can give it a hint though -- by doing an *explicit* type conversion:
```c
*(uint16_t *)p = 0xfeed; // OK
```

Now that we have talked about the basic idea, let's take a look at three different uses you may find in software projects out there.

## Practical Use #1: Generic interface functions, handlers
Imagine a function that you write once and re-use for any type of data afterwards. This sounds pretty simple at first. But how could that be done?

After all, functions have certain *signatures*:

```c
void foo(int);
void bar(float, char);
void baz(double *);
```

Obviously, when calling these functions, you'll need to pass in arguments of the appropriate type. Otherwise, your compiler will either try to implicitly convert the value or raise an error.

In order to make a certain function usable for different data types, you can use parameters of type `void *`. Prominent Example: [qsort](http://man7.org/linux/man-pages/man3/qsort.3.html) from the C Standard Library.

The `qsort` function is for sorting arrays of any type. The following snippet shows its signature:
```c
void qsort(void * base, size_t nmemb, size_t size,
           int (*compar)(const void *, const void *));
```
This actually demonstrates two different uses of void pointers. The function expects `base` to be the start of the array to be sorted. Note that it **doesn't need to know how memory is organized** at that location, hence the void pointer here.

Even more exciting is the 'compar' parameter. It is a *function pointer* to a *comparison function* which expects **two void pointers**. When providing our own comparison function we need to make sure it has exactly this signature.

Inside the function body, we then define the *explicit* type conversion (*cast*) necessary to make use of the two void pointers:
```c
#include <stdlib.h>

uint8_t arr[] = { 2, 3, 1 };

int my_compare(const void * a, const void * b)
{
	uint8_t aa = *(uint8_t *)a;
	uint8_t bb = *(uint8_t *)b;

	return (aa - bb);
}

qsort((void *)arr, sizeof(arr) / sizeof(uint8_t),
	sizeof(uint8_t),
	my_compare);

// arr == { 1, 2, 3 } after qsort
```

This short example shows how the concept is designed: By letting the user provide appropriate type conversions, the library code manages to stay type-independent.

## Conclusion
This concludes the first part already. Here's a thought on the explicit cast needed in the example above: While this is the only way, it is kind of a dangerous thing to do: We explicitly tell the compiler how to interpret a certain block of memory. This assumption isn't checked anywhere else.

Still, this is a great way to write reusable functions in C that can be used for any type of arguments.

In the upcoming two parts of this series, we will cover **two more practical examples** of how to use void pointers. Stay tuned!

## References
- [Type Safety (Wikipedia)](https://en.wikipedia.org/wiki/Type_safety)
- [qsort man page](http://man7.org/linux/man-pages/man3/qsort.3.html)
- [Trending C projects on github](https://github.com/trending/c?since=monthly)
- [What is a void pointer? (Quora)](https://www.quora.com/What-is-a-void-pointer-What-are-the-uses-of-void-pointer)

Image by <a href="https://pixabay.com/users/Pexels-2286921/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1834859">Pexels</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1834859">Pixabay</a>
