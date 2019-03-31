+++
title = "Learning a new programming language: Grokking syntax vs. paradigms"
description = "Being able to write code in a given language does not necessarily mean one is a programmer in it"
tags = [
    "computer-science",
    "python",
    "golang",
    "c",
    "c++",
]
date = "2017-12-31"
categories = [
    "Programming",
]
author = "Dan"
summary = true 
+++

## My journey thus far

I began programming in middle school in BASIC on a Radio Shack TRS-80 ("Trash 80") that ran using two 5.25" floppy disks (to be fair, [you pretty much *had* to learn BASIC in order to use the thing](http://baugues.com/trs-80)). I’m not *that* old, we simply didn’t have much money growing up so I got ancient computers from places like the Goodwill (the kind that were so old and junky people literally gave them away). I was lucky to have a neighbor/babysitter that taught me the foundations of BASIC, and one of the benefits of learning old tech was that the (typically outdated) library books covered everything I was learning!

Fast-forward more than two decades later and my journey has taken me from various flavors of BASIC all the way through HTML, CSS, JavaScript, PHP, Python, C, C++, SQL, C#, and more. Recently I decided to learn Go. I was torn between beginning to learn Rust or Go, but for reasons that I’ll spare you from, I ended up choosing Go.

I watched various Pluralsight courses initially to get up and running, and now I’m reading through [*The Go Programming Language*](http://a.co/hTurKuE) by Donovan & Kernighan (yes, *the* Kernighan from K&R! And yes, I chose this book largely because of this co-author because I enjoyed [*The C Programming Language*](http://a.co/cieatqR)). The hardest part for me when transitioning between languages and when initially learning one is learning to *think* in that language. Go syntax is relatively easy because of my prior programming experience, but learning to think (and code) in Go paradigms will take some time.

## "Pythonic" example

An example from Python is in order since more readers will be familiar with that language.

I learned to write simple for loops in C (and other C-like languages) using this basic pattern:

```
for (i = 0; i < 10; i++) {
    do_something(i);
}
```

When I first learned Python, coming from C and other C-like languages, I wrote for loops like so:

```
i = 0
while (i < 10):
    do_something(i)
    i += 1
```

It works, but it's not *Pythonic*. I understood the language *syntax*, but this wasn’t the *optimal* way to do this in Python. As I learned more, I wrote:

```
for i in range(10):
    do_something(i)
```

And then I later learned that in Python 2, `range` is not a generator function and actually allocates the entire range in memory, and began using `xrange` instead (which is no longer an issue in Python 3, you can just use `range` and you get this optimization by default).

This transition didn’t happen overnight. It also didn’t happen *automatically* (nor auto-*magically* for that matter). I worked my way through multiple tutorials, solved coding challenges, read more experienced people’s code, read books about the language ([*Professional Python*](http://a.co/hZUdsQF) was especially helpful to me after I was comfortable with the basics), asked people for help, and **spent a lot of time coding** (with the last item in this list being the most important). I’ve since taught Python at a local university, which also was an educational experience *for me* as learning how to teach the language to students with no programming background also helped me think through what paradigms are important passing on vs. which just confuse newcomers with little gain.

## Heritage and philosophy

When I started learning BASIC as a kid, I just wanted to use my computer and be able to have fun with it (I made simple games and a program to track my family’s VHS, vinyl, and CD collection; the latter program had no persistent storage &mdash; program interruption meant total data loss!). When I learned HTML, I wanted to make a personal web page (on angelfire, w00t!). When I learned CSS, I wanted to make my web page prettier. When I learned JavaScript, I honestly just wanted to make an infinite loop of alert messages to annoy people. When I learned PHP, it was because I could only afford cheap shared hosting and I had finally determined that storing a password client-side in the page source using JavaScript was a bad idea (no matter how much I obfuscated it, disabled right-click, etc.).

But when I first tried learning C++ (and failed), I became acutely aware that I was missing something. My brain was incapable of grokking the paradigms being taught at the time. It was also the first time I had tried learning a programming language solely from a book. And because of that, I read the preface and introduction to the book. This turned out to be eye-opening: it explained the *rationale* behind C++ and its *evolution* from C. This was the the first time I realized that programming languages have an underlying heritage and subsequent philosophy. Up until that point, I had only attempted to learn new languages because of the (perceived) limitations of the ones I already knew (or *thought* I knew).

I also learned another valuable lesson: get help (from *people*, not just books/videos/tutorials). Unfortunately for me at the time, my parents had bought me my first C++ book at a garage sale and it was out of date. I could not get "Hello, World" to compile, which was incredibly frustrating. I was lucky to know someone who already knew the language (he had flippantly commented that I should learn C++, which was the only reason I knew of its existence as a kid). He pointed out that the language had since implemented namespaces, and to get the example in my book to work I had to add `using namespace std;` to my code (at the time I would not have grasped prefacing the identifiers with their respective namespace, eg., `std::cout`). What I probably would have never figured out on my own was explained to me (at least enough to move on from that example) in less than 5 minutes from someone with experience. I’ve not always had the luxury of people who could help me learn a language, but this is getting much better with [Stack Overflow](https://stackoverflow.com/), Slack communities dedicated to specific programming languages, [CodeMentor](https://www.codementor.io/), etc. (but I used to &mdash; and *still* &mdash; get a little help on IRC from time to time).

If you learned C and then tried learning Java (but knew nothing except C paradigms), you might think Java is far too verbose and needlessly complicated. But that would be because you failed to grasp the object-oriented philosophy underlying Java. Conversely, if you went from Java to C, you’d lament the lack of garbage collection and limitations of structs (but fail to grasp the true *power* and *control* you have). Similarly, if you only knew Python and then tried learning any language without a [global interpreter lock (GIL)](https://wiki.python.org/moin/GlobalInterpreterLock), you might have trouble wrapping your head around concurrency, parallelism, and thread safety (which requires a little knowledge of computer/CPU architecture).

Programming languages are generally created in an attempt to make the programmer’s life easier (with the exception of [esoteric languages](https://esolangs.org/wiki/Esoteric_programming_language)). Understanding *where a language came from and why it was created* goes a long way towards understanding how to think in that language's paradigm(s).

## Making it work… optimally

Fully understanding a language's ancestry and paradigm(s) is not necessary to use it. I don’t mean to insinuate that all Python coders need to [read the source code for CPython](https://github.com/python/cpython), but they should at least [read PEP 20](https://www.python.org/dev/peps/pep-0020/) (type `import this` into the Python interpreter, then `import antigravity`).

But understanding at least some of the heritage and philosophy behind a programming language will help approach problems more optimally in a given language (and perhaps know when a language is or isn’t well-suited for a task). Writing code using a different language’s paradigm may *work* (like my first attempt at a for loop in Python), but don’t stop there. You’re missing out on much more powerful paradigms that will help you more *optimally* solve problems in the language you’re using (and maybe even write less code and make your life easier in the process!).

So here I am, learning Go. [I decided to rewrite a program I helped write in Python for a crypto CTF challenge a while back](https://github.com/danzek/ctf-crypto-challenge), which was a great learning experience (and it’s far from perfect!). But I have much more to learn — and always will. Onward ho until [I grok in fullness!](http://a.co/4dezO9d)

