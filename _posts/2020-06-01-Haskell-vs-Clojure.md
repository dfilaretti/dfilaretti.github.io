---
layout: posts
title: From Haskell to Clojure, and back
excerpt: "During the last couple of years, I've been lucky enough to use both Haskell and Clojure professionally (at different companies) and in both cases it's been an enjoyable and rewarding experience."
---

During the last couple of years, I've been lucky enough to use Haskell and Clojure professionally (at different companies) and in both cases it's been a very enjoyable ride.

In this post I'm going to share my two cents about the experience as well as some of the pain points I faced when transitioning from a Haskell to a Clojure codebase.

## Syntax 

As a LISP dialect, some people dislikes Clojure's syntax because it's "full of parentheses".  Personally, I love it because all code follows a uniform structure...

There are  _literals_:

{% highlight clojure %}
"hello world"  ; a string
99             ; an integer
[1, "foo", 2]  ; a vector
foo            ; a symbol 
+              ; another symbol
{% endhighlight %}

and _operations_:

{% highlight clojure %}
(+ 2 3)              ; adds 2 and 3
(sort [5, 2, 4])     ; sort a vector
(if true "yes" "no") ; evaluate to "yes"
{% endhighlight %}

each following the same structure:

{% highlight clojure %}
(op arg_1 arg_2 ... arg_n)
{% endhighlight %}

and this is _almost_ everything there is to know about syntax. 

What is interesting is that any piece of Clojure code is, in fact, just a regular data structure. 

For example, the code:
{% highlight clojure %}
(add 2 3) ; evaluates to 5
{% endhighlight %}

is a Clojure _list_ containing the symbol `add`, followed by integers `2` and `3` (this is a common concept in LISPs and is called [_homoiconicity_](https://en.wikipedia.org/wiki/Homoiconicity#:~:text=A%20language%20is%20homoiconic%20if,manipulated%20by%20other%20Lisp%20code.)).

Not only this is interesting in its own right, but it has practical advantages, too; the fact that programs are just data structures means they are easy to manipulate, transform and even generate (something called [_metaprogramming_](https://en.wikipedia.org/wiki/Metaprogramming#:~:text=Metaprogramming%20is%20a%20programming%20technique,even%20modify%20itself%20while%20running.)). This is why one of LISP's favourite features are their [_macros_](http://clojure-doc.org/articles/language/macros.html) facilities and the ability they bring to easily extend the language or even even create your own DSL.

Let's face it though: with all the nested parenthesis, Clojure code _will_ eventually get cumbersome to edit. This is where text editor plugins such as [_paredit_](http://danmidwood.com/content/2014/11/21/animated-paredit.html) come to the rescue by helping make sure parenthesis remain balanced while also providing a set of powerful _structural editing_ shortcuts to manipulate code at the _expression level_.
This may feel confusing at first, but stick to it and you'll see how nice and useful it can be!

Haskell, on the other hand, has a more "mainstream" (or shall we say... modern?) syntax. Similarly to Python, it tries to minimise parenthesis by making indentation count when delimiting blocks of code.  

One thing I particularly like about Haskell's syntax is that it tends to look more uncluttered and similar to actual mathematical notation. 
For example, a function `abs` that returns the absolute value of its argument `n` can be defined in Haskell as:

{% highlight haskell %}

abs :: Int -> Int
abs n | n >= 0    = n 
      | otherwise = -n 

{% endhighlight %}

which in my opinion looks a bit more readable that its Clojure counterpart:

{% highlight clojure %}
(defn abs 
  [n]
  (if (>= n 0) n -n))
{% endhighlight %}

<!-- Because Haskell programs are _not_ Haskell data structures, we can't manipulate them as easily as we'd do in Clojure (or other LISPs). 
This doesn't mean that it isn't possible though, bur just that it may not be as easy/elegant as it'd be in a LISP. It is my understanding in fact that metaprogramming in Haskell can indeed be done via _Template Haskell_, a tool in which I have personally never used and therefore I'll abstain from commenting on. -->

## Type System 

One of the main differences between Haskell and Clojure is probably the fact that Haskell is _statically typed_ while Clojure is _dynamically typed_.
<!-- There already exist plenty of flame wars about the subject online, so I will abstrain from putting gasoline on fire. -->

Frankly, I believe that no approach is "better" than the other in absolute terms. 
Both can count on very smart people among their supporters, and while some of us engage in endless online discussions, great software gets written every day using languages in both categories. 

Having said that, I shall mention that most of my experience is in statically typed languages.
The first language I ever learnt was Pascal, followed by C, Java, C#, C++ and, much more recently, Haskell.

It's no surprise then, that as I started familiarising with my first Clojure codebase I found the lack of static typing uncomfortable, and even after being introduced to [_schema_](https://github.com/plumatic/schema) (which essentially allow you to place runtime checks to make sure that data matches a certain "shape"), I could't really shake off the feeling that something was missing.

<!-- In Haskell and other statically typed languages, types are an integral part of the language and eventually becomes second nature. 
On the other hand, tools like schema or clojure.spec not only are _optional_ but, even when adopted, can be selectively turned on or off (e.g. allowing only a selected number of functions or namespaces to be checked).
This certainly introduces extra flexibility, but also can also new opportunity for mistakes (or sloppiness).  -->

<!-- I heard people complain about the overhead of the Haskell type system, and about how dealing with it can sometimes distract you from the actual problem you are trying to solve.
But is working with schemas/specs really much better at the end of the day? Is is really less hassle?
At least in Haskell (and other static languages) you don't have to think about it too much as it's all built in in the language, not to mention all those trivial runtime errors that are automatically rules out at ocmpile time... (and you can always add runtime checks and testing if you want). -->

Maybe it's just me, but In Clojure-land I spent several hours tracking down trivial errors (such as a `nil` being erroneously passed to a function) that just don't happen in Haskell.
Also, I felt that _refactoring_ can be a bit painful.
In Haskell, things such as e.g. swapping the order of a function's parameters tends to be trivial: make a small/incremental change, compile and then fix all the errors (broken call sites etc.). 
In Clojure, unfortunately, we don't have that luxury: you can still make the change, but then is up to you to manually identify all the call sites (or whatever change is required) - an activity that can easily turn error prone and time consuming.

<!-- It certainly happened to me that after a change like that "everything seemed to be working" only to find a day later that indeed something broke along the line.  -->

<!-- I'm sure that you've also heard the aneddote "once Haskell code compiles it almost certianly work" and that "refactoring is cheap".
In my experience, this actually tends to be true.
Make no mistake, there's no magic here; instead of jotting something down and then spending time debugging it, you simply do part of this work upfront when designing your types and the functions operating on them and making sure it all matches nicely.
It may take a few itertions and it can indeed be frustrating, but once it compiles, then you have already made a long way and, yes, quite often it "just works" (TM). -->

It should be clear at this point that my biggest frustration with Clojure at the moment is the lack of a type system (I am aware of [`core.typed`](https://github.com/clojure/core.typed) but I understand it's not widely used/recommended and I've never used it myself - I should really take a look at some point!)

## Tooling & Ecosystem

When I started working in Haskell I searched for "best tools" (IDEs etc.) but couldn't find a one-stop answer; I tried some of the existing solutions with varying degrees of success (and frustration), but eventually [after reading this blog post](https://www.parsonsmatt.org/2018/05/19/ghcid_for_the_win.html) I  settled with a text editor and two terminal windows - one with a REPL and another with [`ghcid`](https://github.com/ndmitchell/ghcid).
This may appear to be a rudimentary setup but I found it surprisingly effective and pleasant to use.

Clojure, on the other hand, seems to benefit from a great variety of tried-and-tested IDEs ([CIDER](https://docs.cider.mx/cider/index.html), [Cursive](https://cursive-ide.com/), [Calva](https://github.com/BetterThanTomorrow/calva) etc.) and various other tools.
The feeling I've got is that while many Haskell programmers are happy to stick to simpler setups (such as the text editor + `ghci` mentioned above), Clojure programmers tend to rely more heavily on IDEs and other tools.
<!-- Those are useful and fun, but they sometimes come with a learning curve, not to mention the potential confusion when trying to figure out which combination of tools to pick in the first place. -->

Something else to be mentioned is that while Haskell compiles to native language, [Clojure is a _hosted language_](https://clojure.org/about/jvm_hosted) based on the _Java Virtual Machine (JVM)_ which has the advantage of providing instant access to the wider Java ecosystem including the multitude of real-world, tried-and-tested libraries and tools.
This can be seen as a positive or as a negative, but there's no doubt it can prove very useful in the right circumstances.

In conclusion, I'd say that while Haskell may appear to be a little behind in terms of tooling and ecosystem, that doesn't necessarily mean a poor development workflow; with a strong emphasis on the type system and on a workflow revolving around a programmer-compiler dialogue, you may find that you don't need super-advanced external tools to be productive in Haskell.

## Learning curve

Haskell has a reputation for being difficult to learn, but I tend to disagree.
It may be true that Haskell is different from the languages that most of us are are used to, but that doesn't necessarily make it harder.

<!-- When I was learning Java in high school, the teacher once said that programming languages are all "the same" and once you know one you can easily switch to another with minor syntax adjustments; I believed her at the time, and I personally held that belief for years.  -->

Some people believe that programming languages are all "the same" and that once you master one it is easy to switch to another with minor syntax adjustments.
This may be true when moving between similar languages such as e.g. Java and C#, but learning a functional language isn't just about a new syntax - it's about learning a different way to think about programming!

Many _"functional first"_ languages such as Scala, F# and even Clojure can make this transition smoother since they offer a mix of functional and Object Oriented features in an attempt to combine "the best of both worlds".

Haskell however is different in that it doesn't try to combine programming paradigms; instead, it embraces the _functional programming_ style in its purest form.
This basically means that there are no shortcuts - if you are new to it, you'll have to step back, grab yourself a good textbook and learn with a beginner mentality, just as if you were learning how to program for the first time.
Once you do however, you'll quickly see that there's nothing to be afraid of, not even the so dreaded _monads_!

<!-- When I decided to do that a couple of years ago, I soon realised that there was nothing too scary about Haskell.
Sure, some of the concepts may appear strange at first but if you approach it with a beginner's mentality and some curiosity there really is nothing to be scared of and eventually it'all makes sense (and you'll be asking yourself how could you stay all those years without it). -->

Clojure, on the other hand, seems to enjoy a somehow opposite reputation. 
I remember reading somewhere about how any developer could learn it "on the job" in a week or two and immediately become productive. 
I don't feel that's entirely realistic either...

The language feels indeed practical and friendly and let's face it - being able to do something like this:
{% highlight clojure %}
(slurp "my-file.txt")             ; read from a file
(slurp "https://www.google.com/") ; read from a URL
(rand)                            ; generate a random number
{% endhighlight %}

without worrying about `IO`, monads or `do` blocks can feel liberating at first.

<!-- However, my learning experience wasn't completely pain free.
After figuring learning some basics, I was still left with questions such as "how does REPL-driven development actually work and how should I structure my code to take advantage of that? How do I make the most of a LISP? How do I write idiomatic code? Should I use editor plugin X or Y? and countless others. -->

<!-- First, I had to face the lack of a type system (discussed earlier); then, I had to figure out how which editor, IDE and tools I wanted to use (which itself took several iterations). 
Then, once I finally had it all working, I had to figure out... well, what to make of it all. 
People celebrate the fast, REPL-driven development experience offered by Clojure, but how does that work in practice? Does having your IDE set up and your REPL up and running mean you are practicing REPL Driven Development or is there more to that? Does the code has to be structueed in a certain wey? And what are the best practices anyway and how do you learn to fully embrace the LISP mentality and write idiomatic code?  -->

But, honestly, I don't believe that Clojure is so much "easier" than Haskell at the end of the day. 
For sure, the fact that it is dynamically typed and that it allows mixing pure and impure code (unlike Haskell) can make it easier for a beginner to get started, but eventually you'll still find yourself wondering:
* how does REPL-driven development actually work? 
* what's the best way to structure my code?
* how do I take advantage of using a LISP and how do I write idiomatic code? 
* how do I make the most of macros? 

and countless others...

Make no mistake: Clojure it's a powerful programming language that comes with a rich (and sometimes confusing) ecosystem and a deep cultural heritage (remember LISP?). 
It may be easy to grasp the basics but if you want to get past that you'll probably still need to set aside some time for some serious learning.

I'm fully aware that different people have different learning styles but, personally, it's only when I started to approach learning Clojure in a more systematic way that the penny finally dropped and I started to really like it.

<!-- It's when I started reading about it (as I've done previously with Haskell) and making an effort to learn it properly that I really started to love Clojure.
What I'm criticising here is not Clojure, but that myth that want people to believe that it's so easy you can just "learn it on the job" in a couple of weeks. 
This may be convenient for companies, but, if you are like me, do yourself a favour, take some time and learn about it and you won't be disappointed (and, by the way there are _excellent_ learning material about Clojure, free or otherwise). -->

## Industries & jobs

Both Haskell and Clojure are _general purpose_ programming languages meaning there's nothing preventing you from adopting them in any given field or domain. 
Having said that, there seem to be certain niches where they tend to get more traction then others.

This may vary depending on your location and network, but, from what I can see (and sadly there aren't tons of Clojure or Haskell jobs around), Clojure gigs tend to be mostly at startups and involve some form of web programming - often "full-stack" with the backend written in Clojure and the frontend in ClojureScript.

<!-- There are a lot of very cool, user friendly, well documented and actively maintained projects to help you with that. An example I'm particularly familiar with is re-frame, a clojure script framework for writing single page applications in clojurescript based on the reactive functional programming paradigm. 
In my experience those things tend to be easy to setup, are well documented actively maintained and there's a very friendly and actve community to refer to in case you need help.  -->

On the other hand, while it is certainly possible to build web applications with Haskell, my impression is that this isn't the primary niche. 
Where haskell seems to be gaining momentum instead is in fields such as finance, blockchain, compilers/interpreters/tools, formal methods etc. (and, in general, niches where safety & correctness are perceived as important).


## Conclusions

Of all the programming languages I've worked with, Clojure and Haskell are without any doubt among my favourites. 

I love Clojure for its LISP heritage, focus on getting things done, and the great community and ecosystem (while, sadly, I can't help but finding the lack of a type system frustrating).
I love Haskell for its conceptual elegance, focus on types and abstractions and for the emphasis on safety and correctness.
Finally, I love both because, ultimately, they are _fun_ and rewarding!

But don't forget: at the end of the day, _programming languages are just tools_... a great codebase written in some "uncool" language is still better than a nasty Haskell or Clojure codebase.
Similarly, a healthy, well-functioning team is better than a toxic and dysfunctional one - even if they turn out to be using the latest/fanciest/best technology.

Happy coding!
