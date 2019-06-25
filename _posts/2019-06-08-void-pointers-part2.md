---
layout: post
title: "3 Practical Uses of void Pointers in the C Language (part 2/3)"
image: "/img/arrows.jpg"
---

In [Part 1]({% post_url 2019-05-07-void-pointers-part1 %}), we talked about how to use void pointers for implementing generic interface functions. Now we're going to see how to take advantage of them to hide implementation details.

## Practical Use #2: Hide implementation details
You can use void pointers to build *Abstract Data Types ([ADT](https://en.wikipedia.org/wiki/Abstract_data_type))*. Why is that something one might want to have, you ask?

An ADT is a type that is defined by its *data* and *operations* on that data. They are *abstract* because they hide internal details from the user. That's essentially what a C++ class provides.

The following snippet shows an example of an ADT in C:

```c
// Counter.h
void * Counter_init(void);
void Counter_increment(void *handle);
uint16_t Counter_getValue(void *handle);
```

What you see is the *public interface* of a counter that can be created, incremented and queried.

- `Counter_init()` returns a handle of type `void *`. By doing it that way, the user doesn't need to make any assumption about internal representation, e.g. how the counter value is stored.
- The *void pointer handle* can be passed to one of the operations, `Counter_increment()` or `Counter_getValue()`.
- You can even create multiple instances of that type (or module) by maintaining a separate handle for each of them. James Grenning provides a great description of this concept in his book [Test Driven Development for Embedded C](https://www.amazon.com/Driven-Development-Embedded-Pragmatic-Programmers/dp/193435662X).

This is how you would use it:
```c
// main.c

void *handle[2] = {};
handle[0] = Counter_init();
handle[1] = Counter_init();

printf("%d\n", Counter_getValue(handle[0]));
printf("%d\n", Counter_getValue(handle[1]));

Counter_increment(handle[0]);

printf("---\n");
printf("%d\n", Counter_getValue(handle[0]));
printf("%d\n", Counter_getValue(handle[1]));
```

Output:
```
0
0
---
1
0
```

This demonstrates the use of two instances of `Counter` that don't interfere with each other. The output shows that only the first one is incremented.

Apart from its interface (the header file), what does actually happen behind the scenes of the module? The corresponding source file provides some insights:
```c
// Counter.c
struct InternalData
{
  uint16_t count;
};

void * Counter_init(void)
{
  void *mem = malloc(sizeof(struct InternalData));
  ((struct InternalData*)mem)->count = 0;
  return mem;
}

void Counter_increment(void *handle)
{
  ((struct InternalData*)mem)->count++;
}

uint16_t Counter_getValue(void *handle)
{
  return ((struct InternalData*)mem)->count;
}
```

On initialization, memory is allocated for an internal data structure that holds the counter's current value. The pointer to that memory location is then returned, without exhibiting any detail about its internal representation.

Admittedly, the internal representation is not a surprise here, of course -- it's an integer value.

### Type Safety
If you look at the code above, what else do you notice?

Well, we are completely on our own when it comes to converting our handle from `void *` to the actual internal type. In the implementation of each operation, we *assume* that the incoming handle points to some memory of the expected type. Because of the cast, the compiler has no way to check whether that assumption holds.

There's nothing stopping us from doing things like that, for instance:
```c
float a = 42;
Counter_increment((void *)&a);
```

Here, the floating-point number is clearly not the same as `struct InternalData`.

The C language provides the concept of an [incomplete type](https://en.cppreference.com/w/c/language/type#Incomplete_types). With that in mind, we may rewrite the Counter module like that:
```c
// Counter.h
struct InternalData; // incomplete type

struct InternalData* Counter_init(void);
void Counter_increment(struct InternalData* handle);
uint16_t Counter_getValue(struct InternalData* handle);
```

The incomplete type `struct InternalData` allows us to use an actual type for the `handle` pointer, instead of *void*. That way the compiler will be able to do the type checking for us.

The implementation is still hidden from the user, only the *incomplete* declaration is visible from the header file.

Sample usage (for a single instance; multiple ones would still work just as before):

```c
// main.c

struct InternalData *handle = Counter_init();
Counter_getValue(handle);
Counter_increment(handle);
```

## Conclusion
The void pointer can be an enabler for Abstract Data Types in C. A safer alternative might be to use incomplete types instead.

In the next and last part of the series, we will introduce yet another practical use of the void pointer in C. Stay tuned!

What is your experience with these techniques? Feel free to send me a twitter message!
