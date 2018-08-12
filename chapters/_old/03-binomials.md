---
layout: chapter
title: Analyzing binomial data
description: "The simplest of data types"
---



~~~~
var model = function() {
  var theta = uniform(0,1) // prior on parameter
  var outcome = binomial( {n:10, p: theta} ) // predict data
  condition(outcome == 5) // condition on data
  return {"theta": theta}
}

var posteriorDist = Infer({method: "rejection", samples: 1000, model: model})

viz(posteriorDist)
~~~~

What does it look like in comparison to the prior?

~~~~
var model = function() {
  var theta = uniform(0,1)
  var thetaPrior = uniform(0,1)
  var outcome = binomial( {n:10, p: theta} ) // predict data
  condition(outcome == 5) // condition on data
  return {"theta": theta, "thetaPrior": thetaPrior}
}

var posteriorDist = Infer({method: "rejection", samples: 1000}, model)

viz.marginals(posteriorDist)
~~~~
