---
layout: chapter
title: Building models
description: "Describing the generative process of data"
---

In the last chapter, we played around with the basic building blocks of probabilistic programs: sampling from and explicitly representing probability distributions.
In this chapter, we will see how to build probabilistic programs that describe the generative process of data: how the observed data could have been generated.
By explicitly articulating this generative process, we can use Bayesian inference to learn about the latent (or, unobservable) scientifically-interesting parameters of this process.

<!-- Drawing inferences beyond our experimental sample is the domain of statistics. 
If we were only interested in the sample we collect in an experiment (e.g., if you happen you sample the entire population of interest), then there is no need for statistics: You just can describe the data and that would be the end of the story.
But that is not our usual situation: We collect experimental data in order to learn something about the population of interest (and in doing so, we often learn about a hypothesis of interest).
 -->

## A simple generative process

Imagine we are investigating the **origins of prosocial behavior**:
Some believe humans are born with a self-interested tendency and only acquire prosociality through instruction (perhaps mediated by language).
thers believe altruism in innate; as soon as children are physically and cognitively able to help, they will.
To investigate this, we are going to put pre-verbal infants in a context where a person is struggling to complete their goal (e.g., as in [this classic study](https://www.youtube.com/watch?v=kfGAen6QiUE)).

We plan to conduct this experiment on 20 infants. 
Each child may or may not help; what we are interested in is the population's *propensity* to help.
A simple generative model of this task would be that infants' propensity to help is like the bias of a coin, and whether or not they help in the moment is the result of flipping that coin. 

~~~~
var propensity_to_help = 0.5
var child_helps = function(){ flip(propensity_to_help) }
viz(repeat(1000, child_helps))
~~~~

This model is said to be "generative" because it generates observable data (`child_helps`) from an underlying latent construct (`propensity_to_help`).
Using the technique known as Bayesian inference, we can reason backwards using this generative process to learn about the latent parameter from observabed data (some number of kids who help).
To do so, we will first have to describe our state of knowledge before we have collected data.
We will also have to further elaborate this generative model to be able to generate data sets of size 20 (for our 20 kids).
<!-- That is, we can ask: Given that we have observed `k` out of `n` children help, what can we say about children's `propensity_to_help`? -->


### Incorporating scientific uncertainty

Above, we simulated a possible outcome (i.e., a child helping vs. not) assuming some `propensity_to_help`.
But this is not super helpful because we do not know the `propensity_to_help`.
In fact, that is the very thing we would like to learn about!
Rather than stipulate `propensity_to_help`, it would be nice to figure out what the parameter *should be* given observed data.
To do this, the Bayesian data analyst must describe their state of knowledge about the latent parameter before having observed any data.

> **Prior distribution**: The state of knowledge of the scientist before any data has been collected.

Sometimes, the *prior distribution* is informed by our expertise as scientists.
Other times, the prior is what we think an objective scientist might assume *a priori* (e.g., a skeptical reviewer).
Determining what the prior distribution should be is one of the intellectual challenges of doing Bayesian data analysis. 
(Indeed, coming to terms with what we truly believe is one of the intellectual challenges of life.) 
Because we do not know what `propensity_to_help` should be, our knowledge must be represented by ... dun dun dun ... a *probability distribution*, which assigns (potentially different) probabilities to different possible values of the parameter.
The task of articulating a prior distribution then reduces to the task of choosing the appropriate distribution and the parameters of that distribution. 
(As you might imagine, there is potentially no end to this process. What should the parameters of the prior distribution be? Well, if you're not totally sure, you might want to articulate a distribution of possible values for the prior parameters, a so-called *hyperprior* distribution. Often, we don't need to be so extreme in our explicit representation of uncertainty, but it is an important tool to remember that you have in your toolkit.)
We will break down this task into constituent parts and in more detail later in chapter.

For our `propensity_to_help`, we know it should be a probability (so, a number between 0 and 1) and the most agnostic belief one might hold is that it *could be any* number between 0 and 1, and there is no reason to prefer some more than others *a priori*.
The Uniform distribution represents this kind of unknowledge.

~~~~
var samplePrior = function(){
  var propensity_to_help = uniform(0, 1)
  return { propensity_to_help }
}

viz(repeat(10000, samplePrior))
~~~~


<!-- The *propensity of 16-month-olds to help* is a latent construct; we scientists cannot observe the propensity directly. 
Instead, the latent propensity to help manifests in children's actual helping behavior; that is, it manifests in the data.
If 16-month-olds have a high propensity to help in our experimental condition, then many of the actual 16-month-olds we test should help. -->


<!-- To learn about the propensity
Suppose we plan to collect 20 kids worth of data (For now, there is no control condition)
Each kid does the single-trial experiment one time, producing either a *helping* (coded as: 1 or `true`) or *not helping* (0 or `false`) behavior. -->

<!-- Note that this sort of binary question: (*do 16-month-olds help?*) is a *hypothesis testing* question.  -->
<!-- Loosely speaking, hypothesis A is that they do help and hypothesis B is that they do not help. -->
<!-- For now, let us put aside the binary question and ask the more continuous question: "What is the propensity of 16-month-olds to help?" -->
<!-- This is still a statistical inference question, since we are not going to measure all 16-month-olds in the world (and even if we could, we still might want to generalize to the 14-month-olds two months in the future...). -->



### Linking to data sets

We can combine this prior distribution over the parameter with our generative process of the data, which we wrote down above.
Composing these together will produce a prior distribution over observed data.
Distributions over observed data are called **predictive distributions** and so this is the *prior predictive distribution*.

~~~~
var samplePriorPredictive = function(){
  var propensity_to_help = uniform(0, 1)
  return {child_helps: flip(propensity_to_help)}
}

viz(repeat(10000, samplePriorPredictive))
~~~~

The support of this distribution (i.e., the values over which this distribution is defined) are the possible outcomes of the experiment, whereas the **parameter distribution** above is defined over the values of the latent parameter.


**Exercises:**

1. How does this distribution differ from the one above it? Why?

2. How does this distribution differ from the codebox above the one above this one? Why?


Now, we said we would collect 20 kids worth of data (every one of the 20 infants does the task exactly once).
So far, we've only considered a single flip.
To constuct a set of 20 data points, we simply flip 20 times.


~~~~
var samplePriorPredictive = function(){
  var propensity_to_help = uniform(0, 1)
  var child_helps = function(){ flip(propensity_to_help) }
  var simulated_data_set = repeat(20, child_helps)
  // summarize data
  var number_of_kids_who_helped = sum(simulated_data_set)
  return {number_of_kids_who_helped}
}

viz(repeat(10000, samplePriorPredictive))
~~~~


In the code box above, we defined a generative process of possible observed data in our experimental setting.
It is a simple generative process, where we assume each participant produces a response *independent* of all others (this is implicit in the `repeat`), and with the same probability (`propensity_to_help`).
Note that this assumption underlies a lot of simple frequentist statistics, like a binomial test. 
We might want to relax the first assumption, if for example, all twenty kids were tested simultaneously in the same location and there was the possibility that behavior of one child would influence the other.
We might relax the second assumption if we have more information about each child that we thought might be relevant and/or that is of scientific interest (e.g., demographic variables that we might be interested in like gender or SES, but also *repeated measurements* of the same child).
We represent the data above (`simulated_data_set`) as an array of Boolean responses (`true` or `false`) as well as the number of `true` (or helpful responses; `number_of_kids_who_helped`).

This particular generative process (independent flips of a coin) is a very common one and actually has its own primitive distribution that corresponds to the generative process: the `Binomial` distribution.
So, we could rewrite the above code as the following:

~~~~
var samplePriorPredictive = function(){
  var propensity_to_help = uniform(0, 1)
  var PredictiveDistribution = Binomial({n: 20, p: propensity_to_help})
  var number_of_kids_who_helped = sample(PredictiveDistribution)
  return {number_of_kids_who_helped}
}

viz(repeat(10000, samplePriorPredictive))
~~~~

Note that using the `binomial` distribution discards the order of the data, which we are assuming is not relevant. (We assume the same model for the 4th participant as we do for the 14th participant.)
It is not always the case that the generative process of the data that we want to assume in our experimental setting has a canonical distribution (like the Binomial) that corresponds to it. But using probabilistic programs, we can represent any generative process by stringing together multiple sampling statements, and possibly transforming them using mathematical (deterministic) operations. 


### Updating beliefs with actual data

Suppose we run the experiment and 15 of the 20 kids help.
Can we use this information to learn about the latent parameter `propensity_to_help`?
It turns out, this is exactly the setting to apply Bayes Rule. 
In WebPPL, this looks like:

~~~~
var model = function(){
  var propensity_to_help = uniform(0, 1)
  var PredictiveDistribution = Binomial({n: 20, p: propensity_to_help})
  observe(PredictiveDistribution, 15)
  return { propensity_to_help }
}

var PosteriorDistribution = Infer({model})
viz(PosteriorDistribution)
~~~~

The posterior distribution represents the scientist's state of knowledge after having observed what came out in the experiment. 
How did we get this?
We added two ingredients to our generative model: `observe()` and `Infer()`.
`observe()` tells the program the data that was observed (above: 15) and the distribution that it was assumed to be generated from (above: `PredictiveDistribution`, a Binomial).
`Infer()` then tells the program to perform Bayesian inference model: in a sense, to integrate the prior and the likelihood with the observed data.

Compare this simple program with the simple form of Bayes Theorem.

$$
P(\theta \mid d) \propto P(d \mid \theta) \times P(\theta)
$$

**Exercise**: Where in the program is the prior represented? Where in the program is the likelihood represented?

The output of `Infer()` is a distribution object like the built-in distributions like `Binomial()` (hence, we capitalize the name in keeping with that convention).
When we ran the code box, some messages were printed out. 
That is because there is no one solution to the problem of performing [Bayesian inference](https://en.wikipedia.org/wiki/Bayesian_inference) (i.e., `Infer()` by itself is underspecified).
WebPPL automatically tries to find a good way to run `Infer()`. That is, it tries to pick a good method or algorithm for inference and it decided upon ["rejection"](https://en.wikipedia.org/wiki/Rejection_sampling) as the algorithm.
Algorithms for inference is a technical topic that will be discussed in a later chapter.
At the end of this chapter, we will describe how Rejection Sampling works.

### Describing posterior distributions

#### Credible intervals

Using Bayesian data analysis, scientists can make probability statements about the underlying parameter. 
For example, you can take the posterior distribution and find the points between which 95% of the probability mass is located. 
This is called a Bayesian credible interval. 
(Note that there are actually several related but different ways of creating credible intervals. citet:kruschke2014doing has a helpful discussion of these.)

The following code computes a credible interval in a crude way.
The code chunk below calls out to an external JavaScript library called "Lodash" using `_.fn()`. For documentation about lodash, see [https://lodash.com/docs/](https://lodash.com/docs/).
(It should be noted: There are better ways of doing this outside of WebPPL. For example, the `coda` package in R can take a list of samples and estimate a smooth curve for them, and then compute a credible interval.
)

~~~~
var credibleInterval = function(mySamples, credMass){
  var sortedPts = sort(mySamples)
  var ciIdxInc = Math.ceil(credMass*sortedPts.length)
  var nCIs = sortedPts.length - ciIdxInc

  var ciWidth = map(function(i){
    sortedPts[i + ciIdxInc] - sortedPts[i]
  },_.range(nCIs))

  var i = _.indexOf(ciWidth, _.min(ciWidth))

  return [sortedPts[i], sortedPts[i+ciIdxInc]]
}


var posteriorSamples = editor.get("posteriorSamples")

credibleInterval(posteriorSamples, 0.95)
~~~~

The interpretation of a Bayesian credible interval is that "There is a 95% chance that the parameter is between X and Y".
This is in contrast to classical confidence intervals, for which probability statements about parameters are verboten. 
This leads to confusion among practitioners: In one study, the vast majority of undergraduates, masters, and PhDs misunderstood the basic interpretation of a *classical confidence interval*, instead importing a Bayesian interpretation to it refp:Hoekstra2014:misinterpretation.

#### Posterior predictives

Just as with the prior, we can consider both the posterior parameter distribution (what we should believe about the `propensity_to_help`) and the posterior predictive distribution (what we should expect to observe in future experiments).
Though we may think of these as different distributions, they are in actuality just two ways of looking at the same information.

~~~~
var model = function(){
  var propensity_to_help = uniform(0, 1)
  var PredictiveDistribution = Binomial({n: 20, p: propensity_to_help})
  observe(PredictiveDistribution, 15)
  return { posteriorPredictive: sample(PredictiveDistribution) }
}

var PosteriorPredictiveDistribution = Infer({model})
viz(PosteriorPredictiveDistribution)
~~~~


### Understanding the full model


#### The 4 Ps: Priors, posteriors, parameters, predictives


~~~~
// observed data
var k = 15 // number of children who help
var n = 20  // total number of children

var model = function() {

   // propensity to help, or true population proportion who would help
   var p = uniform(0, 1);

   // Observed k children who help
   // Assuming each child's response is independent of each other
   observe(Binomial({p : p, n: n}), k);

   // predict what the next n will do
   var posteriorPredictive = binomial(p, n);

   // duplicate model structure and parameters, without observe
   var prior_p = uniform(0, 1);
   var priorPredictive = binomial(prior_p, n);

   return {
       prior: prior_p, priorPredictive : priorPredictive,
       posterior : p, posteriorPredictive : posteriorPredictive
    };
}

var opts = {model: model, method: "rejection", samples: 2000};
var posterior = Infer(opts);

viz.marginals(posterior)
~~~~

The posterior distribution depends upon: (i) the prior distribution, (ii) the assumed generative process of the data, and (iii) the observed data. 
(ii & iii are sometimes referred to collectively as the "likelihood of the observed data").

**Exercises:**

1. Try to interpret each plot, and how they relate to each other. Why are some plots densities and others bar graphs? Understanding these ideas is a key to understanding Bayesian analysis. Check your understanding by trying other data sets, varying both `k` and `n`.

2. Try different priors on `p`, by changing `p = uniform(0, 1)` to `p = beta(10,10)`, `beta(1,5)` and `beta(0.1,0.1)`. Use the figures produced to understand the assumptions these priors capture, and how they interact with the same data to produce posterior inferences and predictions.

3. Predictive distributions are not restricted to exactly the same experiment as the observed data, and can be used in the context of any experiment where the inferred model parameters make predictions. In the current simple binomial setting, for example, predictive distributions could be found by an experiment that is different because it has `n' != n` observations. Change the model to implement an example of this.




















### Abstracting away from the algorithm: `Infer`


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





We can build a probabilistic program to represent a scientific, generative model that can simulate different possible outcomes of our experiment.
(Later, we will use the actual outcome of the experiment to learn about a parameter of this model.)
For our purposes, a generative model is simply one that provides a mapping from latent, unobservable constructs to observable data.
In our example, the latent construct is the *propensity* to help (or, how many 16-months-olds in the whole population would help) and the observable behavior is the *number* of kids in our experiment who help.

The simplest, most idealized model is a `flip()` model: The probability that a kid helps is in proportion to the underlying propensity to help. 
That is, if 16-month-olds have a 0.75 propensity to help, and we collect twenty kids worth of data, we would expect on average 15 of them to help.



**Exercise:** See how changing the `propensityToHelp` changes the observed `numberOfHelpfulResponses`. Make a new function that returns `numberOfHelpfulResponses`. Then use `viz(repeat(1000, newFunction))` to visualize the empirical distribution on `numberOfHelpfulResponses`.

For example, suppose in this experiment children can help to varying degrees (if they decide to help). Suppose that the degree to which a child can help is Gaussian (normally) distributed, and not helping is translated to a helping degree of -2 (arbitrarily). Such a generative model could look like:

~~~~ norun
var singleParticipantModel = function(){
  var help = flip(propensityToHelp)
  var helpingDegree = help ? gaussian(0, 1) : -2
  return helpingDegree
}
~~~~

Incorporating this into our code for generating a full sample worth of data:

~~~~
// define parameters
var numberOfKidsTested = 20
var propensityToHelp = 0.75

var singleParticipantModel = function(){
  var help = flip(propensityToHelp)
  var helpingDegree = help ? gaussian(0, 1) : -2
  return {helpingDegree}
}

var observableResponses = repeat(numberOfKidsTested,singleParticipantModel)

viz.hist(observableResponses)
~~~~

We will see more of this kind of elaboration of models in [Chapter 5](5-advancedBDA.html).

#### Prior distributions over parameters and data


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
// DOES NOT RUN
factor(val)
observe(Dist, val) === factor(Dist.score(val)) === condition(sample(Dist) == val)
condition(bool) === factor(bool ? 0 : -Infinity)
~~~~


In the [next chapter](04-bdaFundamentals.html), we'll go through the fundmantals of Bayesian analysis.