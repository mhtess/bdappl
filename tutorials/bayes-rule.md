# Bayes rule

In this tutorial, we're going to walk through how to get from a prior belief distribution to a posterior belief distribution through the process of observing data.

We'll start with the most canonical prior belief distribution: a uniform distribution over the numbers 0 and 1 (oft interpreted as the weight of a coin, or the true population proportion of doing X in my experiment).

## Prior over parameter


~~~~
var prior = function(){
  var p = uniform(0,1)
  return p
}

viz.density(repeat(10000, prior))
~~~~

## Prior over observed data (prior predictive)

~~~~
var priorPredictive = function(){
  var p = uniform(0,1)
  var outcome = binomial({"n": 20, "p": p}) // linking function
  return outcome
}

viz.hist(repeat(10000, priorPredictive))
~~~~

## Updating beliefs with data

So now we're ready to integrate in our data.
One very simple way of doing this is to say: 

1. Sample a coin weight from our prior `uniform(0,1)`
2. Make a prediction (i.e., sample an outcome), given that coin `binomial( {n:20, p: p} )` 
+ If our prediction matches our data, keep the coin.
+ If our prediction doesn't match our data, throw away the coin.

In principle, any coin weight that's not exactly 0 **could** give rise to our data (with a coin weighted 0.2, it could produce 15 / 20 heads, though it is unlikely). It turns out, if you repeat this procedure many times, then you end up with the correct posterior distribution on coin weights. It is called [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling).

So,
+ Repeat many times

This is an algorithm for doing *Bayesian inference* (going from a prior belief distribution to a posterior belief distribution).


~~~~
var sampleFromPosterior = function(){
  var p = uniform(0,1)
  var outcome = binomial( {n:20, p: p} )
  return (outcome == 15) ? p : -99 
}

// remove all the -99s
var samples = filter(function(s){return s != -99 }, 
                     repeat(100000, sampleFromPosterior))

viz.density(samples)
~~~~

### A useful abstraction

~~~~
var model = function() {
  var p = uniform(0,1) // priors
  var outcome = binomial( {n:20, p: p} ) // predict data
  condition(outcome == 15) // condition on data
  return {"theta": p}
}

var posteriorDist = Infer(
  {method: "rejection", samples: 1000}, 
  model
)

viz.auto(posteriorDist)
~~~~