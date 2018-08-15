---
layout: appendix
title: Bayesian inference in a probabilistic program
description: "Understanding observe, condition, and factor via rejection sampling"
---

#### Learning about parameters from data

Having specified our *a priori* state of knowledge about the parameter of interest, and the generative process of the data given a particular value of the parameter, we are ready to make inferences about the likely value of the parameter given our observed data.
We do this via *Bayesian inference*, the mathematically correct way of reasoning about the underlying probability that generated our data.

So we run the experiment, and 15 out of 20 kids performed the helping behavior.
Thus, `numberOfHelpfulResponses == 15`.
How can we tell our model about this?

~~~~ norun
var sampleAndObserve = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  var matchesOurData = (numberOfHelpfulResponses == 15)
  return ...
}
~~~~

What should we return?
We could return `matchesOurData`.
If we repeat this function many times, we will estimate how many `propensityToHelp`s under our prior (i.e., between 0 - 1) give rise to our observed data.
This is called the **likelihood of the data**, but is not immediately interpretable in isolation (though we will see it later in this course).

What if we returned `propensityToHelp`?
Well, that will just give us the same prior that we saw above, because there is no relationship in the program between `matchesOurData` and `propensityToHelp`.

What if we returned `propensityToHelp`, but only if `matchesOurData` is `true`?
In principle, any value `propensityToHelp` *could* give rise to our data, but intuitively some values are more likely to than others (e.g., a `propensityToHelp` = 0.2, would produce 15 out of 20 successes with probability proportional to $$0.2^{15} + 0.8^5$$, which is not as likely as a `propensityToHelp` = 0.8 would have in producing 15 out of 20 success).

It turns out, if you repeat that procedure many times, then the values that survive this "rejection" procedure, survive it in proportion to the actual *a posteriori* probability of those values given the observed data. 
It is a mathematical manifestation of the quotation from Arthur Conan Doyle's *Sherlock Holmes*: "Once you eliminate the impossible, whatever remains, no matter how improbable, must be the truth."
Thus, we eliminate the impossible (and, implicitly, we penalize the improbable), and what we are left with is a distribution that reflects our state of knowledge after having observed the data we collected.


We'll use the `editor.put()` function to save our results so we can look at the them in different code boxes.

~~~~
var PriorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;

var sampleAndObserve = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  var matchesOurData = (numberOfHelpfulResponses == 15)
  return matchesOurData ? propensityToHelp : "reject"
}

var exampleOutput = repeat(10, sampleAndObserve)

display("___example output from function___")
display(exampleOutput)

// remove all the rejects
var posteriorSamples = filter(
  function(s){return s != "reject" },
  repeat(100000, sampleAndObserve)
)

// save results in browser cache to access them later
editor.put("posteriorSamples", posteriorSamples)

viz(posteriorSamples)
~~~~

Visualized from the code box above is the *posterior distribution* over the parameter `propensityToHelp`.
It represents our state of knowledge about the parameter after having observed the data (15 out of 20 success).

#### The inference algorithm

The procedure we implemented in the above code box is called [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling), and it is the simplest algorithm for [Bayesian inference](https://en.wikipedia.org/wiki/Bayesian_inference).

The algorithm can be written as:

1. Sample a parameter value from the prior (e.g., `p = uniform(0,1)`)
2. Make a prediction (i.e., generate a possible observed data), given that parameter value (e.g., `binomial( {n:20, p: p} )`)
+ If the prediction generates the observed data, record parameter value.
+ If the prediction doesn't generate the observed data, throw away that parameter value.
3. Repeat many times.

Just as we saw in the previous chapter, our ability to represent this distribution depends upon the number of samples we take.
Above, we have chosen to take 100000 samples in order to more accurately represent the posterior distribution.
The number of samples doesn't correspond to anything about our scientific question; it is a feature of the *inference algorithm*, not of our model.
We will describe inference algorithms in more detail in a later chapter.


### Abstracting away from the algorithm with `Infer`


~~~~
var priorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var model = function() {
  var propensityToHelp = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  condition(numberOfHelpfulResponses == 15) // condition on data
  return { propensityToHelp }
}

var inferArgument = {
  model: model, 
  method: "rejection", 
  samples: 5000
}

var posteriorDistibution = Infer(inferArgument)

viz(posteriorDistibution)
~~~~


Intuitively, `condition()` here operates the same as the conditional return statement in the code box above this one. 
It takes in a boolean value, and throws out the random choices for which that boolean is `false`. 
Speaking more generally and technically, `condition()` *re-weights* the probabilities of the *program execution* (which includes all of the *random choices* that have been made up to that point in the program) in a binary way: If it's true, the probability of that program execution gets multiplied by 1 (which has no effect) and if the condition statement is false, the probability of that program execution gets multiplied by 0 (which completely destroys that program execution).

`condition()` is a special case of `factor()`, which directly (and continuously) re-weights the (log) probability of the program execution. 
Whereas `condition()` can only take `true` or `false` as arguments, `factor()` takes a number. 
The code above can be rewritten using factor in the following way:


~~~~
var priorDistribution = Uniform({a:0, b:1});

var model = function() {
  var propensityToHelp = sample(priorDistribution)
  // reweight based on log-prob of observing 15
  factor(Binomial( {n:20, p: propensityToHelp} ).score(15))
  return { propensityToHelp }
}

var posterior = Infer({model: model, method: "rejection", samples: 1000})
viz(posterior)
~~~~

Re-weighting the log-probabilities of a program execution by the (log) probability of a value under a given distribution, as is shown in the code box above, is true Bayesian updating. Because this updating procedure is so commonly used, it gets its own helper function: `observe()`.

~~~~
var model = function() {
  var propensityToHelp = uniform(0,1) // priors
  observe(Binomial( {n:20, p: propensityToHelp} ), 15) // observe 15 from the Binomial dist
  return { propensityToHelp }
}

var posterior = Infer({model: model, method: "rejection", samples: 1000})
viz(posterior)
~~~~


#### Observe, condition, and factor: distilled

The helper functions `condition()`, `observe()`, and `factor()` all have the same underlying purpose: Changing the probability of different program executions. For Bayesian data analysis, we want to do this in a way that computes the posterior distribution. 

Imagine running a model function a single time. 
In some lines of the model code, the program makes *random choices* (e.g., flipping a coin and it landing on heads, or tails).
The collection of all the random choices in an execution of every line of a program is referred to as the program execution.

Different random choices may have different (prior) probabilities (or perhaps, you have uninformed priors on all of the parameters, and then they each have equal probability).
What `observe`, `condition`, and `factor` do is change the probabilities of these different random choices. 
For Bayesian data analysis, we use these terms to change the probabilities of these random choices to align with the true posterior probabilities. 
For BDA, this is usually achived using `observe`.

`factor` is the most primitive of the three, and `observe` and `condition` are both special cases of `factor`. 
`factor` directly re-weights the log-probability of program executions, and it takes in a single numerical argument (how much to re-weight the log-probabilities). 
`observe` is a special case where you want to re-weight the probabilities by the probability of the observed data under some distribution. `observe` thus takes in two arguments: a distribution and an observed data point.

`condition` is a special case where you want to completely reject or rule out certain program executions.
`condition` takes in a single *boolean* argum

Here is a summary of the three statements.

~~~~ norun
factor(val)
observe(Dist, val) === factor(Dist.score(val)) 
                   === condition(sample(Dist) == val)
condition(bool) === factor(bool ? 0 : -Infinity)
~~~~
