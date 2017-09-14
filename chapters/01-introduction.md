---
layout: chapter
title: Introducing probabilistic programs
description: "An introduction to probabilistic programs"
---

If you would like a brief refresher on JavaScript basics, go [here](http://probmods.org/chapters/13-appendix-js-basics.html).

## Sampling

A [probabilistic programming language (PPL)](https://en.wikipedia.org/wiki/Probabilistic_programming_language) is a programming language specialized for building probabilistic models and *performing inference* in those models. What that means, we'll see in a moment. But first, the core ingredient: *random primitives*.

~~~~
flip(0.6)
~~~~

`flip` is a function like any other: it takes an argument and returns a value. What makes `flip` special is that it doesn't do the same thing every time you run it.

Because it is a function (like any other), we can pass it to higher order functions like `repeat()`

~~~~
repeat(100, flip)
~~~~

WebPPL has its own basic visualization tools that are already outfitted in this website. It has a lot of functionality, whose documentation can be found [here](https://github.com/probmods/webppl-viz). The coolest thing about WebPPL viz is that it will often automatically produce exactly what you want to see, just by calling `viz()`. (This functionality used to be called `viz.magic()`).

> **Exercises:**
>
> 1. Try calling `viz()` on the output of the above code box.
> 2. Run the code box several times. Does the output surprise you?
> 3. Try repeating the flip 1000 times, and run the code box several times. What has changed? Why?
> 4. Try repeating `flip(0.6)`. What needs to change from the original code box?

## Distributions

Above, we looked at *samples* from probability distributions. Using functions that sample, the probability distribution is *implicit* in those sampling functions. For example, if you repeat the sampling many times, you will start to approximate the distribution better and better.

WebPPL has the ability to represent probability distributions *explicitly*. This is done using a capitalized versions of the sampler functions. (Note: `flip()` is a cute way of referring to a sample from the `bernoulli()` distribution.)

~~~~
bernoulli(0.6)
~~~~

Distributions have parameters. (And different distributions have different parameters.)
For example, the Bernoulli distribution has a single parameter.
In WebPPL, it is called `p` and refers to the probability of the coin landing on heads.
[Click here](http://docs.webppl.org/en/master/distributions.html) for a complete list of distributions and their parameters in WebPPL.
We can call the distributions by passing them objects with the parameter name(s) as keys and parameter value(s) as values (e.g., `{p: 0.5}`)

~~~~
Bernoulli( { p: 0.5 } )
~~~~

> Exercise: Try running the above box many times. Does it change? Why not?

If a distribution is explicitly constructed, it can be sampled from by calling `sample()` on that distribution.

~~~~
var parameters = { p : 0.9 }
var myDist = Bernoulli( parameters )
sample(myDist)
~~~~

> Exercise: Line by line, describe what is happening in the above code box.

The **support** of a distribution is the set of possible values that can be sampled from that distribution. Distributions' supports can be access by calling `.support()` on the distribution.
The **score** of a value under a distribution is the log-probability of observing that value given that distribution. The score can be access by calling `.score(val)` on the distribution.

~~~~
// the support of the distribution
display( Bernoulli( { p : 0.9 } ).support() )
// the log-probability of true
display( Bernoulli( { p : 0.9 } ).score(true) )
~~~~

## Challenge problem

~~~~
var geometricCoin = function(){
  ...
}

~~~~

> **Exercises:**
>
> 1. Make a function (`geometricCoin`) that flips a coin (of whatever weight you'd like). If it comes up heads, you call that same function and add 1 to the result. If it comes up tails, you return the number 1.
> 2. Pass that function to repeat, repeat it many times, and create a picture using `viz()`

In the [next chapter](02-buildingModels.html), we'll see how to use new information (e.g., observations, data) to update uncertainty in probabilistic programs.
