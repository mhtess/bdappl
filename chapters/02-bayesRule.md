---
layout: chapter
title: Bayes Rule
description: "Updating beliefs with data"
---

In the last chapter, we sampled from distributions and visualized them.
As scientists, we can use distributions to represent our uncertainty about latent parameters of interest (e.g., the true effect size of a manipulation in the population).
Distributions that we specify are called "prior distributions".
Once we have data or observations, we can use those data and ask "What should we believe (about the latent parameters) after having seen this data"?
In this tutorial, we're going to walk through how to get from a prior belief distribution to a posterior belief distribution through the process of observing data.

We'll start with the most canonical prior belief distribution: a uniform distribution over the numbers 0 and 1 (oft interpreted as the weight of a coin, or the true population proportion of doing X in my experiment).

## Prior over parameter

~~~~
var prior = function(){
  var p = uniform(0,1)
  return p
}

viz(repeat(10000, prior))
~~~~

## Prior over observed data (prior predictive)

Let's imagine we ran a "helping experiment" on 20 children.
Children observed some person struggle with completing a task and we see how many of them offer help, like in [this study](https://www.youtube.com/watch?v=kfGAen6QiUE)

*A priori*, what can we say about how many kids will help?
For starters, the number must be between 0 - 20.
Second, in theory, we might think there is some "true population proportion" of kids who would help. (For example, if we tested every toddler in the United States). Without knowing anything else about who we're testing, we may want to assume that each kid helps with probability equivalent to this true population proportion (generally referred to as the "latent parameter").


It's useful to specify this second part (sometimes referred to as the "linking function") because fundamentally, we are interested in making inferences about the true population proportion.

~~~~
var priorPredictive = function(){
  var p = uniform(0,1)
  var outcome = binomial({"n": 20, "p": p}) // linking function
  return outcome
}

viz(repeat(10000, priorPredictive))
~~~~

## Updating beliefs with data

Now we collected data. And found that 15 of our 20 kids helped. How much do we learn about the true population proportion? Put another way, what is the probability that the next toddler we collect will help?

First, let's describe an algorithm.
One very simple way of doing this is to say:

1. Sample a parameter value (i.e., a coin weight) from our prior `uniform(0,1)`
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
  return (outcome == 15) ? p : "throw array"
}

// remove all the "throw arrays"
var samples = filter(function(s){return s != "throw array" },
                     repeat(100000, sampleFromPosterior))

viz(samples)
~~~~

### Abstracting away from the algorithm

As Bayesian data analysts, we are often in the role of going from a prior distribution to a posterior distribution. The rejection sampling algorithm we described above is a universal way of doing this (it will always get you the right answer). However, it might take an extremely long amount of time to do so (for example, if your observed data is statistically unlikely given your priors). Current research in computer science is trying to figure out more and better ways of getting to a posterior distribution.

Probabilistic programming languages provide a useful abstraction barrier that (often) let's the data analyst worry (a little) less about the algorithm to get the posterior. This lets her just specify her priors, her linking function, and the data, and returns to her the posterior. We show this abstraction below.

~~~~
var model = function() {
  var p = uniform(0,1) // priors
  var outcome = binomial( {n:20, p: p} ) // predict data
  condition(outcome == 15) // condition on data
  return {"theta": p}
}

var posteriorDistibution = Infer({
  model: model, method: "rejection", samples: 1000
  })

viz(posteriorDist)
~~~~

From here, I recommend checking out [the BDA chapter](https://probmods.org/chapters/14-bayesian-data-analysis.html) that is part of probmods.
