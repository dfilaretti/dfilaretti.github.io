---
layout: posts
title: Turning C++'s switch statement into an expression
excerpt: "In the last few months I got involved in audio software development with C++ and the JUCE framework, and since at the same time I’ve been writing Haskell in my day job "
---

In the last few months I got involved in audio software development with C++ and the JUCE framework, and since at the same time I've been writing Haskell in my day job (as well as exploring other functional languages on my own) it didn't take me long to become curious about the whole "functional programming in C++" thing.

In this post I summarise a nice little trick I came across recently while [whatching talks](https://www.youtube.com/watch?v=YgcUuYCCV14) and [reading about C++](https://www.amazon.co.uk/Tour-C-Depth/dp/0134997832/ref=pd_lpo_sbs_14_img_0?_encoding=UTF8&psc=1&refRID=0J2C5W50QPMSTGXF833X).

Let's take a look at the following code:

{% highlight cpp %}

enum class ErrorCode { e_1, e_2, e_3 };

std::string msg;
if (errCode == ErrorCode::e_1)
    msg = "This is a fatal error.";
else
    msg = "This is a minor error.";

{% endhighlight %}

Quite common, rigth? Basically, we want to pick one from a set of alternatives to initialise a data structure (here a simple string `msg`), potentially doing different operations in each branch.

This is a simple, artificial example, but we can already see where the problems are:

* the intended purpose of this code is to _initialise_ `msg` depending on `errCode`, however, what it technically does it to initialise it with a default value and subsequently _assign_ the desired value to it. 

* there's nothing preventing us from using the variable `msg` _before_ it gets the desired value; sure, it may not happen in this example, but it could be possible in larger, messy code. 

* even if we aren't going to reassign `msg` anywhere else, the varieble is now declared as mutable.

##  Expressions vs Statements 

The problem is that in C++ (as in most imperative languages) the `if` statement is, indeed, a _statement_ and not an expression.

What this means is that `if` doesn't "return" a value; instead, all it can do is change the program state in some way (for example by assigning a new value to a variable, printing something to the screen etc.). We say that it performs a _side effect_.

But imagine for a moment that `if` was as expression; if that was possible, it would directly evaluate to a value which we could use to initialise `msg` without intermediate steps:

{% highlight cpp %}

const auto msg =
    if (errCode == ErrorCode::e_1)
        "This is a fatal error.";
    else
        "This is a minor error.";

{% endhighlight %}

Isn't this nicer?
Notice that not only it solves all of the problems above, but it also makes the code more compact by allowing us to use `auto` instead of manually specifying the type of `msg`. 

This is how things work "by default" in functional languges. 
For example, this is how our code looks like in Haskell:

{% highlight haskell %}

msg = 
    if errCode == e_1
        "This is a fatal error."
    else
        "This is a minor error."

{% endhighlight %}

So, can we use a similar style in C++? 
For this example the answer is yes, as we could simply use a _ternary_ (or _conditional_) operator: 

{% highlight cpp %}

const auto msg = 
    (errCode == ErrorCode::e_1) 
        ? "This is a fatal error." 
        : "This is a minor error.";

{% endhighlight %}

The only problem with this is that it doesn't scale - what if we have not only two, but more alternatives?

{% highlight cpp %}

std::string msg;
switch (c)
{
case ErrorCode::e_1:
    msg = "Your memory is full.";
    break;
case ErrorCode::e_2:
    msg = "Your computer has crashed.";
    break;
case ErrorCode::e_3:
    msg = "I need a reboot!";
    break;
}

{% endhighlight %}

Now, it is obvious that a ternary operator won't work; nesting them, on the other hand, will probably make the code too messy. 
So, what can we do?

## Wrapping the `switch` statement into a Lambda

One way to rewrite our initialisation code in a more "expression oriented" manner is to wrap the `switch` _statement_ into a lambda function, like this:

{% highlight cpp %}

const auto msg = [&] {
    switch (errCode) {
        case ErrorCode::e_1:
            return "Your memory is full.";
        case ErrorCode::e_2:
            return "Your computer has crashed.";
        case ErrorCode::e_3:
            return "I need a reboot!";
    }
}();

{% endhighlight %}

It looks nice (to me at least!) and solves all of the problems above making our code more compact, self-contained and less error prone.

Also notice that, although it may look like "extra work" at first, this code is actually shorter than its "non-lambda" counterpart (mostly because we got rid of the `break`s).

But what about performance? 
Since we now added some function-calling machinery to our code, it is indeed possible to face a performance cost. 
My understanding however is that compilers should be able to optimise this in such a way that the generated assembly looks pretty much the same as the one generated from the "non-lambda" version. 
I haven't tried this myself though, so please take it with a pinch of salt (and do your own research/measurements if you are working on highly performance-sensitive code!).

## Conclusion

Functional programmig is becoming more and more mainstream;
not only the use of functional programming languages (such as Haskell, OCaml, F# etc.) is increasing, but, at the same time, traditional imperative languages are adopting more and more "functional" features. An example of this is Lambda functions.

This post discusses a simple trick to rewrite your `switch` statements in a more functional, expression-oriented style using a lambda function. Although we used C++, this should apply to other high-level imperative/O.O. languages like C#, Java etc. 

## References/links

* [A Tour of C++ (Second Edition) by Bjarne Stroustrup](https://www.amazon.co.uk/Tour-C-Depth/dp/0134997832/ref=pd_lpo_sbs_14_img_0?_encoding=UTF8&psc=1&refRID=0J2C5W50QPMSTGXF833X) - this topic is briefly discussed in Section 6.3.3 while introducing lambda functions

* [Functional C++ for Fun and Profit by Phil Nash](https://www.youtube.com/watch?v=YgcUuYCCV14) - interesting talk addressing many topics including the one discussed in this post (3:39)


* [Lambda expressions reference](https://en.cppreference.com/w/cpp/language/lambda)