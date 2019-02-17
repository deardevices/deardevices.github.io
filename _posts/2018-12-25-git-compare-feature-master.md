---
layout: post
title: "Git: Compare feature branches to master (Part 1/2)"
---

This article shows how to display changes in Git feature branches that are not yet merged into master.
We are going to find a simple answer for that simple question, starting off with basic `git diff` commands.

Let's say you have built up the following commit tree:
```
---x---A---B master
        \
         C feature
```
There are two branches, *master* and *feature*. While they have two commits in common, *x* and *A*, each of them introduces another commit, *B* and *C*.

## What's on the *feature* branch that's not yet merged into *master*?

If we look at the tree from above, we notice it's the changes introduced by commit *C* we're interested in.
What's the right Git command for showing those?

Let's try `git diff feature master` first. This returns a bit more than we would like to see though: it also shows changes from the *master* branch.
That's obvious if we think about it: The actual difference we'd like to see is between commit *C* and *A*, not between *C* and *B*!

This leads us to `git diff C A`, which yields the expected result. Unfortunately, real commit hashes are not as easy to remember as single letters, such as *A* and *C*.

In this simple example, we know that *A* is the parent of *C*, so we might replace *A* by *feature~*.
In practice though, commit trees tend to look more like this one:

```
---x---A---o---o---o---o---o---B master
        \
         o---o---o---o---o---o---C feature
```

There's another Git command that determines the *common ancestor* of two commits, which is exactly what we want to know here.
The command `git merge-base master feature` conveniently returns *A* (the commit hash).

This is everything we need to answer the question from above. Let's combine the two `git diff` commands we've worked out so far:
```
git diff $(git merge-base master feature) feature
```

Good news is that Git offers a shorthand for exactly this, the **triple-dot diff**:
```
git diff master...feature
```

Note that the `log` sub command has a slightly different semantic. We're going to cover that here in a follow-up article. 

## References
- [Git docs - diff](https://git-scm.com/docs/git-diff)
- [StackOverflow: Difference between double and triple dot diff?](https://stackoverflow.com/questions/462974/what-are-the-differences-between-double-dot-and-triple-dot-in-git-com)
