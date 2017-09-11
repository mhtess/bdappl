---
layout: chapter
title: Bayes Rule
description: "Updating beliefs with data"
---

In the last chapter, we sampled from distributions and visualized them.
As scientists, we can use distributions to represent our uncertainty about latent parameters of interest (e.g., the true effect size of a manipulation in the population).
Distributions that represent our *state of knowledge before collecting data* are called "prior distributions".

Once we have data or observations, we can use those data and ask: What should we believe (about the latent parameters of interest) after having seen this data?
Given the data we observed, and what we believed *a priori*, what should we believe?

In this tutorial, we're going to walk through how to get from a prior belief distribution to a posterior belief distribution through the process of observing data.
We'll start with the most canonical prior belief distribution: a uniform distribution over the numbers 0 and 1 (oft interpreted as the weight of a coin, or the true population proportion of doing X in my experiment).

## Priors

~~~~
var prior = function(){
  var p = uniform(0,1)
  return p
}

viz(repeat(10000, prior))
~~~~

Let's imagine we are running a "helping experiment" on a sample of twenty children from our local children's museum.
In this experiment, children observe a stranger struggle to complete a goal and we see how many of them offer help, like in [this study](https://www.youtube.com/watch?v=kfGAen6QiUE)

*A priori*, what can we say about how many kids will help?
For starters, if are recruiting 20 children and each child either helps or doesn't help, the number should be between 0 - 20.
Second, we can determine how many children we believe *a priori* would help by considering what we believe *a priori* about the "true population proportion" of kids who would help (the latent parameter we'd like to draw inferences about). For example, if we tested every toddler in the United States). Without knowing anything more about our participants, we assume that each kid helps with probability equal to the true population probability.

This second part is important to specify because it provides a link between the latent parameter and the observed data. This mapping is called the *linking function*.

~~~~
var priorPredictive = function(){
  var p = uniform(0,1)
  var outcome = binomial({"n": 20, "p": p}) // linking function
  return outcome
}

viz(repeat(10000, priorPredictive))
~~~~

## Updating beliefs with data

Now we collected data. And found that 15 of our 20 kids helped. How much do we learn about the true population proportion? What should we believe about the latent parameter?

Here is an algorithm for updating beliefs:

1. Sample a parameter value (i.e., a coin weight) from our prior: `uniform(0,1)`
2. Make a prediction (i.e., sample an outcome), given that coin `binomial( {n:20, p: p} )`
+ If our prediction matches our data, keep the coin.
+ If our prediction doesn't match our data, throw away the coin.

In principle, any coin weight that's not exactly 0 **could** give rise to our data (with a coin weighted 0.2, it could produce 15 / 20 heads, though it is unlikely). It turns out, if you repeat this procedure many times, then the coins you are left with form the true posterior distribution on coin weights. It is called [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling).

So,
+ Repeat many times

This is an algorithm for doing *Bayesian inference* (going from a prior belief distribution to a posterior belief distribution).

~~~~
var sampleFromPosterior = function(){
  var p = uniform(0,1)
  var outcome = binomial( {n:20, p: p} )
  return (outcome == 15) ? p : "reject"
}

// remove all the "throw arrays"
var samples = filter(function(s){return s != "reject" },
                     repeat(100000, sampleFromPosterior))

viz(samples)
~~~~

### Abstracting away from the algorithm: `Infer`

As Bayesian data analysts, we are often in the position of wanting to go from a prior belief distribution to a posterior belief distribution. The rejection sampling algorithm we described above is a universal way of doing this (it will always get you the right answer). However, it might take an extremely long amount of time to do so (for example, if your observed data is statistically unlikely given your prior). Current research in computer science is trying to figure out more and better ways of getting to a posterior distribution.

Probabilistic programming languages provide a useful abstraction barrier that (often) let's the data analyst worry less about the algorithm to get the posterior. This abstraction lets the scientist specify her *generative model of the data* (including: priors, linking function), and the data, and returns to her the posterior. In WebPPL, this abstraction is accomplished by the built-in function `Infer()`.

~~~~
var model = function() {
  var p = uniform(0,1) // priors
  var outcome = binomial( {n:20, p: p} ) // predict data
  condition(outcome == 15) // condition on data
  return { theta: p }
}

var posteriorDistibution = Infer({
  model: model, method: "rejection", samples: 1000
})

viz(posteriorDist)
~~~~

The output of `Infer()` is a distribution object, in the same way as the built-in distributions like `Binomial()`.

Intuitively, `condition()` here operates the same as the conditional return statement in the code box above this one. It takes in a boolean value, and throws out the random choices for which that boolean is `false`. Speaking more generally and technically, `condition()` *re-weights* the probabilities of the *program execution* (which includes all of the *random choices* that have been made up to that point in the program) in a binary way: If it's true, the probability of that program execution gets multiplied by 1 (which has no effect) and if the condition statement is false, the probability of that program execution gets multiplied by 0 (which completely destroys that program execution).

`condition()` is a special case of `factor()`, which directly (and continuously) re-weights the (log) probability of the program execution. Whereas `condition()` can only take `true` or `false` as arguments, `factor()` takes a number. The code above can be rewritten using factor in the following way:


~~~~
var model = function() {
  var p = uniform(0,1) // priors
  // reweight based on log-prob of observing 15
  factor(Binomial( {n:20, p: p} ).score(15))
  return { theta: p }
}
~~~~

Re-weighting the log-probabilities of a program execution by the (log) probability of a value under a given distribution, as is shown in the code box above, is true Bayesian updating. Because this updating procedure is so commonly used, it gets its own helper function: `observe()`.

~~~~
var model = function() {
  var p = uniform(0,1) // priors
  observe(Binomial( {n:20, p: p} ), 15) // observe 15 from the Binomial dist
  return { theta: p }
}
~~~~

From here, I recommend checking out [the BDA chapter](https://probmods.org/chapters/14-bayesian-data-analysis.html) that is part of probmods.
