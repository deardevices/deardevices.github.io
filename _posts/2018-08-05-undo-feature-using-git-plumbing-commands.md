---
layout: post
---

Have you ever thought about adding an *undo* feature to your application? This blog article is an attempt to approach that task by utilizing some internals of the Git version control system together with the Memento design pattern. But please, don't take this too seriously :-).

As described in [Pro Git - Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects), Git is defined as a *content-addressable file system*. According to the book, this simply means that at the core of the tool is a **key-value store** where you can put in arbitrary data and get back a unique key that you can use to access that content later on. You can use *plumbing* commands to access those internal features (in contrast to the more high-level *porcelain* commands like `git commit`).

As an experiment, let's build on top of that file system and use it to design a simple undo feature, together with the *Memento* desgin pattern.

### The Memento design pattern
The basic idea of the [Memento pattern](https://en.wikipedia.org/wiki/Memento_pattern) is to be able to save and restore the state of an object (called *Originator*) from the outside -- without breaking its encapsulation.

Let's take a look at an (yet incomplete) example *Originator* Python class:

```python
  def __init__(self):
    self._state = ''

  def set_state(self, state):
    self._state = state

  def get_state(self):
    return self._state

  def create_memento(self):
    # [gather internal state and put into memento object]
    # [return memento object]

  def restore(self, memento):
    # [restore internal state based on memento object]
```

Note that a client of the *Originator* class (*Caretaker*, as it's called in the pattern) can create snapshots of the internal state without depending on how that state is actually maintained. One **application of the pattern** is an undo concept: By storing a memento object after every change to the *Originator*, we can step back in time quite easily.

### Implement Memento using the Git file system

Now let's go back to Git's core concept -- the content-addressable file system. How can we utilize it for our purpose of saving and restoring the state of an object?

Don't you think the unique key that we get back for each stored data block **looks very much like the content of a memento object?** What if the *Originator* stored its internal state into Git's key-value store and returned the key to the client? For restoring, it would then receive a Memento object containing a key and ask Git for the corresponding state data.

Let's start working on that idea by filling in the missing parts of the *Originator*, assuming we already have some *key_value_storage* that behaves like Git internally:

```python
class Memento:
    def __init__(self, key):
        self.key = key


class Originator:

  # [...]

  def create_memento(self):
    key = self.key_value_storage.store(
        type(self).__name__ + '|' + self._state)
    return Memento(key)

  def restore(self, memento):
    self._state = self.key_value_storage.load(
        memento.key).split('|')[1]
```

The data block that represents the internal state will then be the *_state* attribute, a string in the simplest case, preceded by the name of the class, separated by a `|` character. The *Memento*'s only job here is to store the key returned from the storage.

The next step is to find an implementation for the *key_value_storage*. Its *store* and *load* operations can be fulfilled by Git's  *hash-object* and *cat-file* plumbing commands under the hood, so let's just execute them as a process:

```python
class GitStorage:
    def store(self, value):
        process = Popen(
            ['git', 'hash-object', '-w', '--stdin'], stdout=PIPE, stdin=PIPE)
        result = process.communicate(value.encode('utf-8'))
        return result[0].strip().decode('utf-8')

    def load(self, key):
        process = Popen(['git', 'cat-file', '-p', key], stdout=PIPE)
        return process.communicate()[0].decode('utf-8')
```

That way we store every state as a new Git *blob object*. You can verify the object creation by taking a look at the `.git/objects` folder:

```bash
`find .git/objects -type f | xargs ls -lh --sort=time`
```

Here you may observe that a new hash is created only if the *Originator*'s state has actually changed.

### Conclusion & References

Although this approach looks very promising, we need to take into account how Git is actually supposed to be used: as a **version control system**. In that sense, creating *blob* objects is only a part of the overall procedure of storing a snapshot of a repository. One thing it will certainly interfere with at some point is Git's *garbage collector* (`git help gc`). The reason for that is that the blob objects created are **not referenced by any commit** in the database. 

You can find a working example of the code mentioned in the article at [github/ronalterde/git_memento](https://github.com/ronalterde/git_memento/).

Feel free to send me your ideas on how to make this experimental concept a bit more ready for production.

