---
layout: post
title: "Git: Compare feature branches to master (Part 2/2)"
---

In the [previous article]({% post_url 2018-12-25-git-compare-feature-master %}) we've seen how to ask Git for changes introduced by a particular branch. This time, instead of changes, we would like to know which *commits* were introduced by a branch -- that is, the *hashes* assigned to them.

To figure out how to do that, let's take a look at the first example from last time:
```
---x---A---B master
        \
         C feature
```

There are two branches, *master* and *feature*, sharing a common ancestor, commit *A*. In order to display the entire commit tree, we may go with:
```
$ git log --oneline --graph feature master
```

This shows a tree of all commits reachable from any of the two branches:

```
* 963751a C
| * a3a5c6c B
|/
* a8cb146 A
* 854ccf9 x
```

## What's been introduced by the *feature* branch?

As stated above, we are interested in the commit(s) introduced by the *feature* branch only, commit *C* in this case. The `git log` command works a bit differently compared to `git diff`. The filter expression for the `log` command feels more like a [set operation](https://en.wikipedia.org/wiki/Set_(mathematics)), in a mathematical sense. Here's a simple expression that reads *show commits contained in feature but not in master*:

```
$ git log feature ^master
* 963751a C
```

To visualize what just happened, we can use a [Venn diagram](https://en.wikipedia.org/wiki/Venn_diagram):

![]({{ "/assets/gitlogvenn.png" | absolute_url }})

It shows two sets, one for each branch, the area matching the criteria from above being marked in blue.

## Summary & References
For visualizing `git log`'s filter expressions, it helps to visualize the repo as one or more sets of commits. Venn diagrams turn out to be useful for visualizing the result of operations applied to them.

- [Stack Overflow Article about this issue](https://stackoverflow.com/questions/462974/what-are-the-differences-between-double-dot-and-triple-dot-in-git-com)
