---
layout: post
title:  "Thoughts on Python from a C++ developer’s viewpoint: Style"
---

This article is about Python style. Compared to the C++ world, there are pretty rigid conventions implemented in the language itself as well as in the Python ecosystem. That makes this topic kind of surprising and worth writing some lines about.

<!--more-->

<h2>Blocks</h2>
One of the most obvious things about the language -- as you probably have noticed already -- is the lack of any token for block separation. Instead of having a pair of curly braces for introducing a new code block, simply the <em>level of indent</em> is used.

I still find this surprising sometimes, especially because not even mixing spaces and tabs is allowed in recent versions of the language. Apparently though, people got used to it and seem to utilize the advantages of this strict style requirement. Consider the following C++ snippet:

```cpp
if (flag == true)
    foo();
    bar();
baz();
```

As the indent levels suggest, the author wants both <em>foo()</em> and <em>bar()</em> to be called depending on <em>flag</em>. Unfortunately, C and C++ compilers do not care about indent, which leads to unexpected behavior. In this case, only the call to <em>foo()</em> depends on the value of <em>flag</em>. Both <em>bar()</em> and <em>baz()</em> get called <strong>unconditionally</strong>. 

By the way, this is exactly what made the famous <a href="https://nakedsecurity.sophos.com/2014/02/24/anatomy-of-a-goto-fail-apples-ssl-bug-explained-plus-an-unofficial-patch/">goto fail</a> bug possible. Such a flaw couldn't have happened in Python. Here's the equivalent to the example from above:

```python
if flag == True:
    foo()
    bar()
baz()
```

Here, the behavior perfectly matches the author's intention.

<h2>Style Guide</h2>
If you look at some Python projects <a href="https://github.com/search?o=desc&q=language%3APython+language%3APython">out there</a>, you may notice that many of them adhere to a common style -- aside from the indent convention mentioned above, which is integrated into the language anyway. The style guide described in <a href="https://www.python.org/dev/peps/pep-0008/">PEP 8</a> seems to be widely adopted.

One thing I do not understand about this guide though is the convention for method names. It is stated that all methods must have lowercase names, where words are separated by underscores, e.g. <em>do_some_work()</em>. Compared to camel case (like <em>doSomeWork()</em>) i find this rather hard to write. Maybe it's a bit easier to read, but I'm not sure about that yet.

<h2>Support in Vim</h2>
The Vim text editor has out-of-the-box support for Python block indents. With <em>filetype plugin indent on</em>, it adheres to the rule <em>four spaces, no tabs</em> for all <em>.py</em> files you edit.

Is there anything I've missed? Feel free to leave me a <em>comment</em> or contact me on twitter: <a href="https://twitter.com/ronalterde">@ronalterde</a>.
