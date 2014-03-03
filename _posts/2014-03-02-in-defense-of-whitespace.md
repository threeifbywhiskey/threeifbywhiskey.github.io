---
layout: post
title: "In Defense of Whitespace"
description: ""
categories: 
comments: false
tags: []
---
{% include JB/setup %}

Not that kind of whitespace‒forgive the title bait, but the bikeshedders can lower their hackles now. No, I'm talking about [Whitespace](http://compsoc.dur.ac.uk/whitespace/){:target="new"}, the esoteric programming language. Whitespace programs are composed entirely of spaces, tabs, and newlines. (And you thought Lisp didn't have any syntax!) Despite this ostensible dearth of linguistic capacity, the language is Turing-complete, and I'll go about proving that later in this post. For now, though, I'd like to go into why I think every programmer should familiarize themself with this strange and fascinating language.

As described more thoroughly [here](http://compsoc.dur.ac.uk/whitespace/tutorial.php){:target="new"}, the language is written using a set of 24 instructions‒each encoded as some combination of whitespace characters‒that operate on a stack and a heap, and perhaps you can already see where I'm going with this: programming in Whitespace is very much like writing assembly code. Some programmers find themselves wanting to better understand how computation works at the very bottom, and while assembling doesn't get you *all* the way there, it takes you pretty close. Still, writing "real" assembly can be a painful and tedious process, what with all the architectural anomalies, varying syntaxes, and the plethora of slightly different operations available; writing pseudo-Assembly meant to compile down to Whitespace is practically a pleasure by comparison.

To that end, I've written and made available [a Whitespace assembler](https://github.com/threeifbywhiskey/blacktime){:target="new"}, and the names of the supported instructions can be discerned [here](https://github.com/threeifbywhiskey/blacktime/blob/master/blacktime.c#L5-L6){:target="new"}. For those with whom the idea resonates, coding in Whitespace is a challenging and rewarding process, and I urge every programmer reading to give it a shot. Below, I'm going to present my process of writing a recursive factorial procedure to give you a taste of what it's like to program in this language.

First, we're going to need to read in the number whose factorial we're going to print. Rather than simply reading it onto the stack, Whitespace uses the heap for storing user input. The `ichr` and `inum` instructions read character and numeric input respectively; these instructions pop an "address" from the top of the stack and use it as a key into the heap for storing (and later retrieving) the inputted data. We only need one piece of data, so we'll store it logically enough at address zero.

{% highlight asm %}
push 0
dup    ; The stack now contains zero twice.
inum   ; Now the stack is [0] and the heap is {0: n}.
load   ; And now the stack is [n].
{% endhighlight %}

Now that we have the number on the stack, we're going to call our Fibonacci procedure, print the result and a newline, and then exit.

{% highlight asm %}
call 0
onum
push 10
ochr
exit
{% endhighlight %}

Here we'll define the procedure that gets called above; this is the sort of "backwards" thinking that often goes into writing a Whitespace program. First, we need to check to see whether our number is less than 3; if it is, it's its own factorial and we can return.

{% highlight asm %}
0:
    dup    ; We need a copy of the number to do some math with.
    push 3
    sub    ; The stack is now [n n-3].
    jn 1   ; If the result is negative, jump to 1, a label from which we'll
           ; return. Note that jumping pops its argument, so the stack is [n].
{% endhighlight %}

Now we'll perform the actual factorial calculation, which is to get `n-1` onto the stack, calculate its factorial, and obtain its product with `n`.

{% highlight asm %}
    dup
    push 1
    sub    ; The stack is [n n-1].
    call 0 ; Call factorial on n-1.
    mul    ; Once we've returned, multiply the top two stack elements.
{% endhighlight %}

This is the label we planned to jump to earlier. It simply returns, but this combination of comparisons and gotos is how to do flow control in Whitespace.

{% highlight asm %} 
1:
    ret
{% endhighlight %}

It often makes sense to group a sequence of instructions together on the same line to better convey intent. The following is a logically compacted version of the program:

{% highlight asm linenos %}
push 0 dup inum load 
call 0 onum
push 10 ochr exit

0:
    dup push 3 sub jn 1 
    dup push 1 sub  
    call 0 mul   

1:
    ret
{% endhighlight %}

As far as I'm aware, only [Blacktime](https://github.com/threeifbywhiskey/blacktime){:target="new"} is capable of parsing and assembling this syntax, but there is [prior work](http://web.archive.org/web/20110916050014/http://yagni.com/whitespace/whitespace-asm){:target="new"} in this particular domain. Wayne's Ruby assembler also supports comments and convenience labels, and [his site](http://archive.is/Ge8QD){:target="new"} was largely the catalyst behind my interest in Whitespace. As for interpreters, there is of course the canonical [Haskell one](http://compsoc.dur.ac.uk/whitespace/download.php){:target="new"}, Wayne's [Ruby version](http://web.archive.org/web/20110914162106/http://yagni.com/whitespace/whitespace){:target="new"}, and‒my personal favorite‒Pavel Shub's [C++ interpreter](http://pavelshub.com/blog/tag/whitespace/){:target="new"}. I've written my own [interpreter in C](https://github.com/threeifbywhiskey/satan){:target="new"}, but I haven't quite gotten call semantics down just yet.

I mentioned earlier that I'd go about proving that Whitespace is Turing-complete, and to do that I'll present my [brainfuck interpreter](https://github.com/threeifbywhiskey/blacktime/blob/master/misc/brainfuck.asm){:target="new"}. Given that brainfuck is Turing-complete, this proof by simulation is enough for Whitespace to hold the same distinction. There are plenty more reasonably interesting Whitespace programs in the Blacktime repository, and I again urge anybody with some interest in learning assembly to give Whitespace a try first. It's definitely not the same as writing hyper-optimized tight loops in x86, but it's great mental exercise for the right kind of person, and I hope you're one of them.
