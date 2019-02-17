---
layout: post
title:  "Thoughts on Python from a C++ developer’s viewpoint: Interfaces"
date:   2018-01-07
---
In this episode I‘d like to highlight yet another Python fact I find 
particularly surprising. This article is about *the* design tool in Software Engineering: interfaces.

<!--more-->

We use interfaces in *C++* in the form of abstract base classes. They are the method of choice when it comes to decoupling classes from each other and <a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle">inverting source code dependencies</a>.

The following class diagram shows an example where the class *Foo* depends on *Bar*, but only indirectly through the interface *IBar*. This design enables us to easily change the concrete implementation of *IBar*, e.g. we can put in a *MockBar* object for unit testing.

<img src="{{ site.baseurl }}/assets/classes.png" alt="" width="500" height="230" class="aligncenter size-full wp-image-625" />

Now let's see how to do this in Python. By exploring existing code out in the wild, I've noticed that interface-like concepts are not as as widely used as I had expected. Usually, people create specific classes and pass (references to) them into other classes. There is no base class needed for this to work, no explicit polymorphism. As long as the injected object has the appropriate method to be called, everything  works just fine. In literature, this concept is referred to as <a href="https://en.wikipedia.org/wiki/Duck_typing">duck typing</a>, according to "If it walks like a duck and it quacks like a duck, then it must be a duck.".

Surprisingly, there is not even an interface declaration necessary. This code is perfectly valid:

```python
class Foo:
    def __init__(self, bar):
        bar.quack()
```

Everything that has a method *quack()* is sufficient as a dependency, like this one:

```python
class Bar:
    def quack(self):
        print("quack!")
```

```python
bar = Bar()
f = Foo(bar)
```

While I think this is a nice approach that keeps things simple, for talking about APIs this seems to be a bit too relaxed. How can we specify a common structure all concrete implementations shall adhere to?

Interestingly, this functionality has been added -- not to the language but to Python's standard library. You can find out more about that extension in the article <a href="https://dbader.org/blog/abstract-base-classes-in-python">Abstract Base Classes in Python</a> by Dan Bader.

What is your experience with interface-like approaches in Python? Is there anything I've missed? Feel free to leave me a *comment* or contact me on twitter: <a href="https://twitter.com/ronalterde">@ronalterde</a>.
