---
layout: post
title: "Git in a nutshell: Commit, Checkout, Reset"
image: "/img/ccr.svg.png"
---

We're going to talk about three basic git commands in this episode: `commit`, `checkout` and `reset`. Instead of making up any abstractions or metaphors, we will focus straight on how these commands modify your repository's internal state.

The state of a Git repository is, on the highest level, represented by a chain (or a graph) of *commits*. That's exactly what the `git log` command presents to you, for example.

Commits can be divided even further. For instance, each of them contains a reference to a *tree* object which in turn will eventually end up referencing one or more *blob* objects.

In a previous article ([From Subversion to Git: Snapshots]({% post_url 2018-06-23-from-svn-to-git-snapshots %})), we did an exploration of those. For now, the concept of a commit is sufficiently granular to reason about the three basic commands we are going to explore.

So let's get started!

Here's the state of a sample repo that contains an initial commit already:

```
o 55cd47f Initial Commit  <--- master  <--- HEAD
```

The repo has one branch, *master*, pointed to by *HEAD*. The numbers on the left (the commit hash) serve as a unique identifier for the initial commit.

# 1) git commit
We can use `commit` to create a chain of commit objects. After applying the command two times, our repo will look like that:
```
o 55cd47f Initial Commit
|
o 6411a56 Second Commit
|
o 78bae34 Third Commit <--- master <--- HEAD
```

Each of these commit objects (or just *commits*) represents a snapshot  of the entire repository, i.e. a certain state of the file system.


# 2) git checkout
In the previous listings, *HEAD* always pointed to the latest commit on the master branch.

To jump back and forth between commits, we can use `checkout`. This will *detach* HEAD from the master branch and bring our working copy to the state stored in the commit we are jumping to. In other words: that commit will be *checked out*.

Here's what happens after `git checkout 6411a56`:
```
o 55cd47f Initial Commit
|
o 6411a56 Second Commit <--- HEAD
|
o 78bae34 Third Commit <--- master
```

Only *HEAD* changed. Apart from that, our working copy has been updated accordingly: it now represents the state we captured when creating commit *6411a56*.

# 3) git reset
We've seen how to manipulate the *HEAD* pointer using `checkout`. Similarly, we can use `reset` to modify where branches point to.

Starting with the previous state of the repo, we can move master to the second commit using `git reset --hard 6411a56`:
```
o 55cd47f Initial Commit
|
o 6411a56 Second Commit <--- master <--- HEAD
|
o 78bae34 Third Commit
```

What happened? By `reset`ting, we essentially moved the `master` pointer to the second commit. Here it gets apparent that `reset` can be quite a dangerous operation: the 'Third Commit' is now not referenced at all (i.e. branch or tag). This, in turn, means that it is subject to deletion by git's garbage collector (`git gc`).

# Conclusion
We have seen how to use three basic git commands to manipulate the state of a sample repository. We have talked about how to think about them in terms of the graph of commits maintained by git.

Refer to [Pro Git](https://git-scm.com/book/en/v2/) to find out more!
