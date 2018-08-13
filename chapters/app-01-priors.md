---
layout: appendix
title: Coming up with priors
description: "Systematically interrogating one's knowledge"
---


### Appendix Chapter 1: Prior distributions over parameters and data


Recall that distributions provide both a set of possible values of a variable and their associated probabilities. 
So we can break the problem down into articulating the set of possible values and then the probabilities of each of the values. 

1. **Possible values the parameter could take on**: This is dictated by the role of the parameter in the model.
For example, in abstract terms, `propensityToHelp` is the weight of a coin, and thus must be a number between 0 and 1. (It would not make sense, in the generative model we've described so far, for `propensityToHelp` to be a negative number, or greater than 1.)

2. **Probabilities of all of the possible values of the parameter**: This is much less obvious, but a good starting point is to consider all of the possible values equally probable. That is, we might be completely ignorant as to what the parameter should be (or, we might imagine a host of skeptical reviewers, some of whom think the parameter ought to be high, others who think the parameter ought to be low, and so we intuitively average across all of these potential skeptics). Your understanding of the prior probabilities of the parameter values will grow as you internalize the model more and as we explore the implications of different prior distributions for the results.

So for `propensityToHelp`, a relatively uncontroversial assumption would be that it could be any number between 0 and 1, with all numbers equally likely.
Now that we've made explicit our prior beliefs, we need to find a probability distribution that has these features (if one does not exist, we will need to construct one).
Fortunately, one does exist! 
The [Beta distribution](http://docs.webppl.org/en/master/distributions.html#Beta) is a distribution over the numbers between 0 and 1. 
The Beta distribution has two parameters (denoted in WebPPL as `a` and `b`). 
As can be gleaned from the [Wikipedia article](https://en.wikipedia.org/wiki/Beta_distribution), the parameters that correspond to equal probability for all values between 0 and 1 are `a=1` and `b=1`. 
So our prior distribution is the `Beta({a: 1, b:1})`.

Because assigning equal probabilities to all possible values of a distribution is a very common practice, this state of knowledge can also be described with another family of distributions: The Uniform distribution. The Uniform distribution has two parameters as well (also denoted in WebPPL as `a` and `b`). These parameters are the lower and upper bounds of the range of values. And so, the `Beta({a: 1, b:1})` is equal to `Uniform({a: 0, b:1})`.

~~~~
var PriorDistribution = Uniform({a:0, b:1})

var samplePrior = function(){
  var propensityToHelp = sample(PriorDistribution)
  return {propensityToHelp}
}

viz(repeat(10000, samplePrior))
~~~~

The `Uniform` distribution (bounded between 0 and 1) represents our *a priori* state of knowledge about `propensityToHelp`.
Specifying what we believe *a priori* is the first step towards determining what we should believe *a posteriori*, or after we observe the data.

We can combine this prior distribution over the parameter with our generative process of the data, which we wrote down above.
Composing these together will produce a prior distribution over observed data.
Distributions over observed data are called **predictive distributions** and so this is the *prior predictive distribution*.
The support of this distribution (i.e., the values over which this distribution is defined) are the possible outcomes of the experiment, whereas the **parameter distribution** above is defined over the values of the latent parameter.

~~~~
var PriorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var samplePriorPredictive = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  return {numberOfHelpfulResponses}
}

viz(repeat(10000, samplePriorPredictive))
~~~~

