---
layout: chapter
title: Introducing probabilistic programs
description: "An introduction to probabilistic programs"
---

If you would like a brief refresher on JavaScript basics, go [here](http://probmods.org/chapters/13-appendix-js-basics.html).

## Sampling

A [probabilistic programming language (PPL)](https://en.wikipedia.org/wiki/Probabilistic_programming_language) is  specialized for building probabilistic models and *performing inference* in those models. What that means, we'll see in a moment. But first, the core ingredient: *random primitives*.

~~~~
flip(0.6)
~~~~

`flip` is a function like any other: it takes an argument and returns a value. 
What makes `flip` special is that doesn't return the same value every time you run it, even with the same arguments.
Try running the code box above multiple times to see this.
Flip essentially flips a coin whose probability of landing on heads is given by the parameter value (above: 0.6).
(You can treat the value of `true` as "heads" and `false` as "tails").

Because `flip` is a function (like any other), we can pass it to higher-order functions like `repeat()`
(Recall that "higher-order functions" are functions that take other functions as arguments.)

~~~~
repeat(100, flip)
~~~~

If you are uncertain about the arguments to `repeat()`, or other WebPPL functions, pop over to the [WebPPL docs](http://docs.webppl.org/en/master/functions/arrays.html#repeat) to get it all straight.

### A brief aside for visualization

WebPPL (as accessed in the browser) provides basic visualization tools. 
WebPPL-viz has a lot of functionality, the documentation for which can be found [here](https://github.com/probmods/webppl-viz). 
The coolest thing about WebPPL-viz is the default `viz()` function, which will take whatever you pass it and try to construct a reasonable visualization automatically. 
(This used to be called `viz.magic()`).

> **Exercises:**
>
> 1. Try calling `viz()` on the output of the above code box.
> 2. Run the code box several times. Does the output surprise you?
> 3. Try repeating the flip 1000 times, and run the code box several times. What has changed? Why?
> 4. Try repeating `flip(0.6)`. What needs to change from the original code box? (Hint: Recall that `repeat()` wants to take a function as an argument.)

## Distributions

Above, we looked at *samples* from probability distributions. 
The probability distribution was *implicit* in those sampling functions. 
For example, by repeating the sampling many times, you will start to approximate the distribution better and better.

WebPPL also represents probability distributions *explicitly*. 
We call these explicit probability distributions: **distribution objects**.
Syntactically, this is denoted using a capitalized versions of the sampler functions. 

~~~~
// bernoulli(0.6) // same as flip(0.6)
Bernoulli( { p: 0.6 } )
~~~~

(Note: `flip()` is a cute way of referring to a sample from the `bernoulli()` distribution.)

> Exercise: Try running the above box many times. Does it change? Why not?

Distributions have parameters. (And different distributions have different parameters.)
For example, the Bernoulli distribution has a single parameter.
In WebPPL, it is called `p` and refers to the probability of the coin landing on heads.
[Click here](http://docs.webppl.org/en/master/distributions.html) for a complete list of distributions and their parameters in WebPPL.
We can call the distributions by passing them objects with the parameter name(s) as keys and parameter value(s) as values (e.g., `{p: 0.6}`).
When a distribution is explicitly constructed, it can be sampled from by calling `sample()` on that distribution.

~~~~
var parameters = {p: 0.9}
var myDist = Bernoulli(parameters)
sample(myDist)
~~~~

> Exercise: Line by line, describe what is happening in the above code box.

### Properties of distribution objects

We just saw how you can call `sample()` on a distribution, and it returns a value from that distribution (sampled according to the its probability).
Distribution objects have two other properties that will be useful for us.

Sometimes, we want to know what possible values could be sampled from a distribution.
This is called the **support** of a distribution and it can be accessed by calling `myDist.support()`, where `myDist` is some distribution object.

~~~~
// the support of the distribution
Bernoulli( { p : 0.9 } ).support() 
~~~~

Other times, we have a sample from distribution and we want to know how probable it was to get that sample.
This is sometimes called "scoring a sample", and it can be accessed by calling `myDist.score(mySample)`, where `myDist` is a distribution object and `mySample` is a value. 
Note that for the score of sample to be defined, the sample must be an element of the support of the distribution.
WebPPL returns not the probability of `mySample` under the distribution `myDist`, but the natural logarithm of the probability, or the log-probability.
To recover the probability, use the javascript function `Math.exp()`.


~~~~
// the log-probability of true (e.g., "heads")
Bernoulli( { p : 0.9 } ).score(true)
~~~~


> **Exercises:**
>
> 1. Try changing the parameter value of the Bernoulli distribution in the first code chunk (for the support). Does the result change? Why or why not?
> 2. Try changing the parameter value of the Bernoulli distribution in the second code chunk (for the score). Does the result change? Why or why not?
> 3. Modify the second code chunk to return the probability of `true` (rather than the log-probability). What is the relation between the probability of `true` and the parameter to the Bernoulli distribution?


## Challenge problem

~~~~
var geometricCoin = function(){
  ...
}

~~~~

> **Exercises:**
>
> 1. Make a function (`geometricCoin`) that flips a coin (of whatever weight you'd like). If it comes up heads, you call that same function (`geometricCoin`) and add 1 to the result. If it comes up tails, you return the number 1.
> 2. Pass that function to repeat, repeat it many times, and create a picture using `viz()`

In the [next chapter](02-buildingModels.html), we'll start to compose generative models of data, and use observations (e.g., data we've collected in an experiment) to learn about the models we've created.
