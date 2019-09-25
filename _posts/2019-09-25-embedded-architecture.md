---
layout: post
title: "Software Design Challenges: Embedded vs. Web"
---

Sometimes, Embedded is considered a very special field of Software Engineering. It's all about microcontrollers and complex devices, connected via a plethora of different bus systems -- carefully orchestrated using only the C programming language. Let's go and find out what's so special about Embedded, and what our discipline has in common with, say *Web Development*.

![Printed Circuit Board](/assets/pcb.jpg)

## Introduction
Web and Embedded are two different areas of Software Development, that's for sure. They are very different in terms of programming languages and user interfaces. Also, the application domain is usually not the same; just try to compare a hotel booking platform to a battery-driven GPS logging device.

When it comes to Software Engineering though, Web and Embedded developers are facing **similar problems to solve**. Both usually have to deal with external dependencies that are difficult to set up, expensive, or even both: think of large databases, custom hardware boards, and cloud services.

Handling these dependencies in the smartest way possible is one of the main goals of Software Architecture. There are other goals, of course. What's good architecture? I like the way Robert C. Martin (a.k.a. Uncle Bob) puts it in his book [Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164): A measure of design quality is the effort needed for building *and maintaining* a software system.

In the course of this article, we will first point out things that are specific to each of the two disciplines. After that, it's time to look for commonalities in these two areas.

## Embedded Specifics
In Embedded, it's all about dealing with microcontroller peripherals like timers and input/output facilities. Most of the time, there are some external devices connected to microcontrollers as well, e.g. displays, memory chips or input switches. They all have different timing requirements that need to be fulfilled -- and taken care of by the programmer.

Also very specific to the Embedded domain is the **Target Hardware Bottleneck**, described by James Grenning in his book [Test-Driven Development for Embedded C](https://www.amazon.com/Driven-Development-Embedded-Pragmatic-Programmers/dp/193435662X). The moment we start our programming efforts, final hardware design is often not yet available. There are many reasons for this. One might be that hardware is being developed in parallel. In that case, software developers need to rely on some early (and often rare) prototypes of the hardware.

Real-time requirements are very common in Embedded. When processing some input, it's sometimes necessary to be able to respond within a certain amount of time. That time might as short as some microseconds, or as long as several seconds. The point is that we need to guarantee some process to finish within that time.

With regards to Software Architecture, we tend to aim at architectures that allow us to exchange certain hardware components without us having to re-write everything. It is very common for hardware to change often -- and software needs to keep up.

## Web Specifics
In Web development, programmers usually deal with a ton of other things: databases, Javascript frontends, secure transmission, non-deterministic communication channels (a.k.a. The Internet).

Here, change happens for various reasons, e.g.:
- The database infrastructure is migrated to a different technology.
- There is a brand new and fancy frontend framework we want to base our work on.
- Communications need to be secured.

Clearly, web development today involves managing complex ecosystems.

## What they have in common
After thinking about differences, let's switch focus and see what the common aspects are that these two disciplines share.

I'm assuming that every software program contains a large part of code that is independent of all the things mentioned above. Maybe it's not a distinct module or layer in your program. Maybe it's somewhere in between database API calls or mixed with some auto-generated code. For sure, there is some code that makes up the essence of your program: the **Business Logic**.

I believe focussing on Business Logic is key to understanding how similar Web and Embedded actually are. Of course, low-level microcontroller peripherals are fun to explore and utilize. The same probably goes for the latest and greatest JavaScript frontend framework. Still, most of your knowledge and expertise about your application domain is in the core of your program, captured inside the Business Logic.

It's [Uncle Bob](https://blog.cleancoder.com/) who calls the web a *detail*. That's actually a very good summary of what was described above. It's the Business Logic of your program you should focus on.

## Conclusion
We focussed on two disciplines of Software Engineering that look very different at first sight. After looking at the differences, we came to the conclusion that it is, in fact, Architecture and Design bringing those disciplines close together.

Each discipline has its own set of nitty-gritty details -- the Business Logic being always at the core of the system.
