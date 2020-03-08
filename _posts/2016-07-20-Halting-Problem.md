---
layout: posts
title: The Halting Problem
excerpt: "Yet another Halting Problem blog post."
---

Before looking at the _halting problem_, it is worth considering why it is interesting in the first place.
One of the reasons the halting problem is important is that it is an entry point to a more fundamental issue: _is there anything that computers cannot do_?

A couple of clarifications are in order:

* this is not about processor speed, memory size or other technological details. We are not asking "is there anything that compters cannot do _right now_ but may be able to do in 50 years?". We are asking: "is there anything no computer can do -- doesn't matter if now or in 50 or 1000 years?".

* this is not about things such as "can computers eat and digest ice cream?". There are no tricks: in this post we will be talking about plain, dumb computer-as-a-calculator stuff: reading a numeric or textual input, doing some work and eventually returning some answer (or performing a set of actions).

Before answering this questions, let us first take a quick look at two concepts that will be useful for our understanding of the halting problem and its consequences.

# Terminating programs

What does it mean for a program to terminate?
Well, as the word says, _a program terminates when it finishes its job (whatever that is) in a finite amount of time_ (usually providing an answer or performing some action as a result):

* a program receiving two integers, performing their sum and returning the result to the user. 

* an mp3 converter taking a raw WAV audio file as input, performing some work and after a minute or so returning a compressed mp3 file.

* a search engine receiving a string (the search query), and returning a web page containing the search result.

In reality however, _programs may occasionally not terminate_.
For example, the following function starts printing the string `"hi"` and never stops:

{% highlight python %}

def alwaysLoops ():
    while true:
        print "hi"

{% endhighlight %}

Run it, and your programming environment will likely crash or become unresponsive in some way or another.

This is clearly an extreme example; in reality, this kind of behavior appears sporadically, most likely as a result of some particular input:

{% highlight python %}

def sometimesLoops (n):
    if n == 0:
        while true:
            print "hi"
    else:
        print "DONE"

{% endhighlight %}

Here, we see that `sometimesLoops` goes into an infinite loop only when the input is `0` and prints the string `"DONE"` in all other cases.

While `alwaysLoops` never terminates, `sometimesLoops`  terminates _most of the times_; still, we can't say that it terminates _on all inputs_, because there is one particular input, `0`, for which it does _not_ terminate.

# Programs that eat programs

We have seen programs that takes integers, strings or some sort of data as their input in order to produce their output.
But this is not the end of the story. One thing programs can do, for example, is to _take another program_ as their input, and even produce a new program as their output.

This should be no surprise for programmers since tools like _compilers_, _interpreters_, and _static analysers_ are nothing but programs whose task is to take another program as their input and either: translate it into another form (compiler), execute it (interpreter) or scan it for possible issues (static analyser).

Since programs are stored as plain text files, it shouldn't be surprising to know that there's nothing preventing a program to read another program and do something with it:

{% highlight python %}

def readAProgram (filename):
    program = Path(filename).read_text()
    # now process 'program' in some way!

{% endhighlight %}

This is a huge topic and we'll not go into details here. For now, just bear in mind that as well as taking "data" as their input, program may, in some circumstances, receive, manipulate and return other programs.

# The halting problem

Given an aribitrary program `P`, does `P` _always_ terminate? Or, in other words, does `P` terminate _on all inputs_?
For very simple programs, answering this question is trivial. For example, for both of our examples, `alwaysLoops` and `sometimesLoops` the answer is a _no_.
The reason is that `alwaysLoops` never terminates, while `sometimesLoops` terminates in most cases but it still gets stuck in an _infinite loop_ when the input is `0`.
Therefore, we can claim that none of them terminates _on all inputs_.

So far so good, but what if our program suddently becomes hunderds, thousands or million lines long? 
I'm sure you don't need me to convice you that _our simple reasoning technique will be impractical in this new scenario_.
But... wait! _Maybe we can write a program for that, right?_ That is: can we write a program that takes another program `P` as its input, and tries to determine whether `P` always terminate (or not)?
 
Let's think about it. Suppose such an algorithm, called `termination`, exists; it takes a _file name_ and returns `true` if the program contained in the file terminates (on all inputs) and `false` otherwise:

{% highlight python %}

def termination (filename):
    # 1) read program contained in 'filename'
    # 2) do something smart.
    # 3) return true/false

{% endhighlight %}

So far so good. Remember, we are assuming that that our `termination` function is smart enough so that it can take _any_ program and respond to the halting question correctly.
Consider the following program then (assuming it is stored in `strangeFunction.py`):

{% highlight python %}

def strangeFunction(): 
    while (termination("strangeFunction.py")):
        # doesn't matter what's here!
    
{% endhighlight %}

Does `strangeFunction` terminate? We have two cases to consider:

* _`strangeFunction` terminates_ -- In this case, the call `termination(strangeFunction.py)` must return `true`. Why? Because `termination` takes an existing program and determines whether it terminates or not -- that's its job! Now, take a closer look at the code for `strangeFunction`: can you see where the problem is? That's right: `termination(strangeFunction.py)` returns true (since `strangeFunction` terminates) but this causes the `while` loop to loop indefinitely meaning that... `strangeFunction` _cannot_ terminate!

* On the other hand, assume _`strangeFunction` does not terminate_. Once again `termination(strangeFunction.py)` will do its job correctly, this time returning `false`. But we are up for more troubles: given how `strangeFunction` is deﬁned, this time the `while` loop will finish immediately (since its argument is `false`) which means that _`strangeFunction` does actually terminate_!

Since in both cases we reached a contradiction, our assumption about the existence of `termination` must have been wrong. Therefore, we must take a step back and accept the fact that _a program such as `termination` cannot exists_. 

In other words we have shown that _it is not possible to write a program that solves the halting problem_!

# Conclusion

In this post, we explained the _halting problem_ in an intuitive way and, on the side, we also clarified what we mean by terminating/non-terminating programs and briefly introduced the concept of _programs that manipulates other programs_.

We have seen that despite it being quite a well defined input-output problem (i.e. no "tricks" involved), **it is not possible to write a program that solves the halting problem**.

More broadly, this means that _there exist some problem that cannot be solved by a computer_, which answers our initial question: "is there anything that computers cannot do?" The answer is _yes_, and the halting problem is one example.

So we can't solve the halting problem with a computer -- fair enough. But _is there anything else computers can't do?_ Or is it the halting problem alone?"
This will be the topic of a future post where we discuss how the halting problem affects (among many other things) the branch of computer science that deals with software verification and static analysis.