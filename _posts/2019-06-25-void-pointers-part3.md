---
layout: post
title: "3 Practical Uses of void Pointers in the C Language (part 3/3)"
---

[Part 2]({% post_url 2019-06-08-void-pointers-part2 %}) was all about encapsulation, Abstract Data Types and how void pointers may help you implement these concepts. In this third and last part, we are going to see *yet another use* of them. After that, we will go and find out how void pointers are used in *popular open source projects out there*.

## Practical Use #3: Don't care about the actual type
Sometimes it is not really important what's the type of data a pointer is referring to.

If you think about requesting heap memory using `malloc()` for example, that function is not concerned with what you wish to store at that memory location. Its only job is to give you a contiguous block of memory of a certain size (in bytes). That fact is reflected in its signature, which reads `void *malloc(size_t)`.

It returns a pointer that we may cast to the desired type:
```
struct MyType *foo = (struct MyType *) malloc(sizeof(struct MyType));
```

Another good example is `memcpy`, also a standard function, we can use to copy data between two memory regions. It takes two void pointers and a size as arguments. The concrete type of data is none of its business.

## An Investigation: How others do it
A bit of research on github gives us a good impression on how some popular projects make use of void pointers. Let's focus on *systemd*, *glibc* and the *Linux Kernel*, see what they do and try to map that to the three practical uses we established before.

### systemd
[Systemd](https://www.freedesktop.org/wiki/Software/systemd/) is a modern Linux init system. Well, actually much more than that -- but that's another topic. It is based on C which makes it a good candidate for this research.

There is a mechanism in systemd for parsing configuration data. Presumably because there are so many different types of configuration (more than 200) they employ a list of handlers that share the exact same signature:

```c
int config_parse_bool(void *data);
int config_parse_route_table(void *data);
int config_parse_gateway_onlink(void *data);
/* Note: only parts of signatures are shown here. */
```

Each handler has a void pointer parameter (among some others) that is converted to some concrete type (e.g. bool) inside the function. This looks similar to what we have identified as **Use #1 (Generic Interfaces)**.

### GNU C library
The [glibc](https://www.gnu.org/software/libc/) provides lots of examples for using void pointers, some of which we have already looked at in the course of this blog series.

A good example for **Use #3 (Don't care about data type)** is `fread()`, which is a library function for reading binary data from a file:
```c
size_t fread (void *data, size_t size, size_t count, FILE *stream);
```

To use it, we essentially provide a pointer to a memory region and how many bytes we expect to be stored there. The function implementation is not concerned with the type of that data.

### Linux Kernel

One of the functions inside of some part of the PCI driver has a `void *` parameter called *userdata*:
```c
void pci_walk_bus(int (*cb)(struct pci_dev *, void *), void *userdata);
/* Note: only a part of the signature is shown here. */
```

This clearly is an application of **Use #3 (Don't care about data type)**. The driver code is not concerned with that data in any way.

The only purpose of that pointer is to be passed back to the user as soon as the callback (in parameter `cb`) is called.

## Conclusion
In this blog series, we established three different uses of void pointers in the C language.

Can you think of anything else? Feel free to get in touch via twitter!
