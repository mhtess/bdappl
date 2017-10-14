---
layout: chapter
title: Building models
description: "Describing data and parameters with probabilities"
---

In the last chapter, we played around with the basic building blocks of probabilistic programs: sampling from, and representing, probability distributions.
In this chapter, we will see how to construct distributions that represent *observable outcomes*.

As a running example, we will consider a standard, simple case of data analysis in a developmental psychology experiment: Imagine we are investigators of the **origins of prosocial behavior**, and are testing children to better understand the conditions under which they will provide help to a stranger (for example, as in this classic study where children observe a stranger [struggle to complete a goal](https://www.youtube.com/watch?v=kfGAen6QiUE)
).
In our experimental setting, we will collect 20 kids worth of data in a single condition of an experiment.
Each kid does the experiment one time, producing either a *helping* (1 or `true`) or *not helping* (0 or `false`) behavior.

### Modeling possible observations

We can use distributions to represent the *number of kids who help* in our experiment.

~~~~
var probabilityOfHelping = 0.5, numberOfKidsTested = 20;
var observableResponses= repeat(
  numberOfKidsTested,
  function(){return flip(probabilityOfHelping)}
)
var numberOfHelpfulResponses = sum(observableResponses)
var observedProportionHelping = numberOfHelpfulResponses/numberOfKidsTested;
print(observableResponses)
print("number of kids who helped = " + numberOfHelpfulResponses)
print("proportion of kids who helped = " + observedProportionHelping)
~~~~

In the code box above, we defined a **generative process** of possible observed data in our experimental setting.
It is a simple generative process, where we assume each participant produces a response *independent* of all others (implicit in `repeat`), and with the same probability (`probabilityOfHelping`).
We would want to relax the first assumption, if for example, all twenty kids were tested simultaneously in the same location and there was the possibility that behavior of one child would influence the other.
We could relax the second assumption, if we have more information about each child that we thought might be relevant (e.g., demographic variables that we might be interested in like gender or SES, but also *repeated measurements* of the same child).

We represent the data above as an array of Boolean responses (`true` or `false`) as well as the number of `true` (or helpful responses) and the proportion out of 20.

> Exercise: See how changing the `probabilityOfHelping` changes the observed `numberOfHelpfulResponses`. Wrap the lines code into a function, and have it return `numberOfHelpfulResponses`. Use `viz(repeat(1000, newFunction))` to visualize the *distribution* on numberOfHelpfulResponses.

This particular generative process is a very common generative process and actually has its own distribution that corresponds to the generative process: the `Binomial` distribution.
So, we could rewrite the above code as the following:

~~~~
var probabilityOfHelping = 0.5, numberOfKidsTested = 20;
var numberOfHelpfulResponses = binomial({
  p: probabilityOfHelping,
  n: numberOfKidsTested
})
var observedProportionHelping = numberOfHelpfulResponses/numberOfKidsTested;
print("number of kids who helped = " + numberOfHelpfulResponses)
print("proportion of kids who helped = " + observedProportionHelping)
~~~~

Note that using the `binomial` distribution loses the information available in `observableResponses` in the code box above. The only additional information in `observableResponses` was the order of the data, which we are assuming is not relevant.

It is not always the case that the generative process of the data that we want to assume in our experimental setting has a canonical distribution that corresponds to it.

### Modeling scientific uncertainty

Above, we looked at the possible outcomes of our experiment *given* some `probabilityOfHelping`.
How do we decide what `probabilityOfHelping` to assume?
It is a parameter to the distribution that generates the observed data, but it is *latent* (or, unobservable).
The proportion of kids who help in the experiment (`observedProportionHelping`) is an estimate of this latent parameter, but as you can verify above, it is not the same thing as `probabilityOfHelping`.

As scientists, we are very often in the situation of having observed some data in our *sample* and trying to say something about the underlying, generating probability.
That is, we are trying to make *inferences* from the sample to the population.
In Bayesian terms, we are trying to describe the latent probability.

*A priori*, we don't know the `probabilityOfHelping`.
If children are sensitive to the cues in the our helping experiment, it could be high.
But if children don't care about the stranger in our experiment, it could be 0.5, or it could be close to zero.
Figuring out the value of the variable `probabilityOfHelping` will help us make inferences about the children's sensitivity to our experimental setting, which is ultimately what we're interested in.
How do learn about `probabilityOfHelping`?

#### Prior distributions over parameters and data

First, we explicitly represent our state of knowledge about `probabilityOfHelping` *a priori*.
Distributions that represent our *state of knowledge before collecting data* are called "prior distributions".

~~~~
var priorDistribution = Uniform({a:0, b:1});

var samplePrior = function(){
  var probabilityOfHelping = sample(priorDistribution)
  return probabilityOfHelping
}

viz(repeat(10000, samplePrior))
~~~~

The `Uniform` distribution (bounded between 0 and 1) represents our  *a priori* state of knowledge about the `probabilityOfHelping`.
This is the first step towards learning about scientific questions from data; we must specify what we believe *a priori*.

We can combine this prior distribution over the parameter with (our beliefs about) the generative process of the data, which we wrote down above. 
Composing these together will produce a prior distribution over observed data.
This is called the *prior predictive distribution*, and the support of this distribution (i.e., the values over which this distribution is defined) is in "data space".

~~~~
var priorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var samplePriorPredictive = function(){
  var probabilityOfHelping = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: probabilityOfHelping,
    n: numberOfKidsTested
  })
  return numberOfHelpfulResponses
}

viz(repeat(10000, samplePriorPredictive))
~~~~

> Exercise: How does this distribution differ from the one above it? Why?

#### Constructing posterior distributions

Having specified our *a priori* state of knowledge about the parameter of interest, and the generative process of the data given a particular value of the parameter, we are ready to make a *backwards inference* about the likely value of the parameter given our observed data.
This process is called *Bayesian inference* and represents the mathematically correct way of inductive reasoning about the underlying probability that generated our data.

So we ran the experiment and 15 out of 20 kids performed the helping behavior.
Thus, `numberOfHelpfulResponses == 15`.
How can we tell our model about this?

~~~~ norun
var sampleAndObserve = function(){
  var probabilityOfHelping = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: probabilityOfHelping,
    n: numberOfKidsTested
  })
  var matchesOurData = (numberOfHelpfulResponses == 15)
  return ...
}
~~~~

What should we return?
We could return `matchesOurData`.
If we repeat this function many times, we will estimate the proportion of `probabilityOfHelping` under our prior (i.e., between 0 - 1) that give rise to our observed data.
This is called the **likelihood of the data**, and isn't by itself immediately interpretable (though we will see it later in this course).

What if we returned `probabilityOfHelping`?
Well, that will just give us the same prior that we saw above, because there is no relationship in the program between `matchesOurData` and `probabilityOfHelping`.

What if we returned `probabilityOfHelping`, but only if `matchesOurData` is `true`?
In principle, any value `probabilityOfHelping` **could** give rise to our data, but intuitively some values are more likely to occur than others (e.g., a `probabilityOfHelping` = 0.2, would produce 15 out of 20 successes with probability proportional to $$0.2^{15} + 0.8^5$$, which is not as likely as a `probabilityOfHelping` = 0.8 would have in producing 15 out of 20 success).

It turns out, if you repeat that procedure many times, then the values that survive this rejection procedure, survive it in proportion to the actual *a posteriori* probability of those values given the observed data. This procedure is called [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling), and it is the simplest algorithm for [Bayesian inference](https://en.wikipedia.org/wiki/Bayesian_inference).

The algorithm can be written as:

1. Sample a parameter value from the prior (e.g., `p = uniform(0,1)`)
2. Make a prediction (i.e., generate an outcome), given that parameter value (e.g., `binomial( {n:20, p: p} )`)
+ If the prediction generates the observed data, record parameter value.
+ If the prediction doesn't generate the observed data, throw away that parameter value.
3. Repeat many times.

~~~~
var priorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var sampleAndObserve = function(){
  var probabilityOfHelping = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: probabilityOfHelping,
    n: numberOfKidsTested
  })
  var matchesOurData = (numberOfHelpfulResponses == 15)
  return matchesOurData ? probabilityOfHelping : "reject"
}
var exampleOutput = repeat(10, sampleAndObserve)
print("___example output from function___")
print(exampleOutput)

// remove all the "throw arrays"
var posteriorSamples = filter(
  function(s){return s != "reject" },
  repeat(100000, sampleAndObserve)
)

viz(posteriorSamples)
~~~~

Visualized from the code box above is the *posterior distribution* over the parameter `probabilityOfHelping`.
It represents our state of knowledge about the parameter after having observed the data (15 out of 20 success).

Just as in we saw in the previous chapter, our ability to represent this distribution depends upon the number of samples we take.
Here, we have chosen to take 100000 samples in order to more accurately represent the posterior distribution.
The number of samples doesn't correspond to anything about our scientific question; it is merely an artifact of the inference algorithm.

### Abstracting away from the algorithm: `Infer`

As Bayesian data analysts, we are often in the position of wanting to go from a prior belief distribution to a posterior belief distribution. The rejection sampling algorithm we described above is a universal way of doing this (it will always get you the right answer). However, it might take an extremely long amount of time to do so (for example, if your observed data is statistically unlikely given your prior). Current research in computer science is trying to figure out more and better ways of getting to a posterior distribution.

Probabilistic programming languages provide a useful abstraction barrier that (often) let's the data analyst worry less about the algorithm to get the posterior. This abstraction lets the scientist specify her *generative model of the data* (including: priors, linking function), and the data, and returns to her the posterior. In WebPPL, this abstraction is accomplished by the built-in function `Infer()`.

~~~~
var priorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var model = function() {
  var probabilityOfHelping = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: probabilityOfHelping,
    n: numberOfKidsTested
  })
  condition(numberOfHelpfulResponses == 15) // condition on data
  return { probabilityOfHelping }
}

var posteriorDistibution = Infer({
  model: model, method: "rejection", samples: 1000
})

viz(posteriorDistibution)
~~~~

The output of `Infer()` is a distribution object, in the same way as the built-in distributions like `Binomial()`.

Intuitively, `condition()` here operates in the same way as the conditional return statement in the code box above this one. It takes in a boolean value, and throws out the random choices for which that boolean is `false`. Speaking more generally and technically, `condition()` *re-weights* the probabilities of the *program execution* (which includes all of the *random choices* that have been made up to that point in the program) in a binary way: If it's true, the probability of that program execution gets multiplied by 1 (which has no effect) and if the condition statement is false, the probability of that program execution gets multiplied by 0 (which completely destroys that program execution).

`condition()` is a special case of `factor()`, which directly (and continuously) re-weights the (log) probability of the program execution. Whereas `condition()` can only take `true` or `false` as arguments, `factor()` takes a number. The code above can be rewritten using factor in the following way:


~~~~
var priorDistribution = Uniform({a:0, b:1});

var model = function() {
  var probabilityOfHelping = sample(priorDistribution)
  // reweight based on log-prob of observing 15
  factor(Binomial( {n:20, p: probabilityOfHelping} ).score(15))
  return { probabilityOfHelping }
}
~~~~

Re-weighting the log-probabilities of a program execution by the (log) probability of a value under a given distribution, as is shown in the code box above, is true Bayesian updating. Because this updating procedure is so commonly used, it gets its own helper function: `observe()`.

~~~~
var model = function() {
  var probabilityOfHelping = uniform(0,1) // priors
  observe(Binomial( {n:20, p: probabilityOfHelping} ), 15) // observe 15 from the Binomial dist
  return { probabilityOfHelping }
}
~~~~

From here, I recommend checking out [the BDA chapter](https://probmods.org/chapters/14-bayesian-data-analysis.html) that is part of probmods.
