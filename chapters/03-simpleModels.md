---
layout: chapter
title: Learning about a hypothesis
description: "Models formalize hypotheses"
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
Some scientists believe humans are born with a self-interested tendency and only acquire prosociality through instruction (perhaps mediated by language).
Others believe altruism in innate; as soon as children are physically and cognitively able to help, they will.
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

// save results in browser cache to access them in later codeboxes
editor.put("PosteriorDistribution", PosteriorDistribution)

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
///fold:
var getSamples = function(Dist, key){
  return _.map(_.map(PosteriorDistribution.samples, "value"), key)
}
///

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

var PosteriorDistribution = editor.get("PosteriorDistribution")

var posteriorSamples = getSamples(PosteriorDistribution, "propensity_to_help")

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

You can think about the posterior predictive in a sequential manner. 
First, perform Bayesian inference to learn about the parameter of interest.
Then, using the posterior distribution over the parameter, sample according to the generative model of the data.
This gives you a posterior predictive distribution in "data space"; what we would expect should we run the experiment again.

Using WebPPL, we can see this sequential manner:

~~~~
var model = function(){
  var propensity_to_help = uniform(0, 1)
  observe(Binomial({n: 20, p: propensity_to_help}), 15)
  return propensity_to_help
}

var PosteriorParameterDistribution = Infer({model, method: "rejection",
                                           samples: 1000})

var posteriorPredictiveSamples = repeat(10000, function(){ 
  var propensity_to_help = sample(PosteriorParameterDistribution)
  return binomial({n: 20, p: propensity_to_help})
})

viz(posteriorPredictiveSamples)
~~~~


### The 4 Ps: Priors, posteriors, parameters, predictives

So far, we have seen priors and posteriors, parameters and predictives.
It can be helful to see the 4 Ps in the same model, and to examine each distribution at the same time.

~~~~
// observed data
var k = 15 // number of children who help
var n = 20  // total number of children

var PriorDistribution = Uniform({a: 0, b: 1});

var model = function() {

   // propensity to help, or true population proportion who would help
   var propensity_to_help = sample(PriorDistribution);

   // Observed k children who help
   // Assuming each child's response is independent of each other
   observe(Binomial({p : propensity_to_help, n: n}), k);

   // predict what the next n will do
   var posteriorPredictive = binomial(propensity_to_help, n);

   // duplicate model structure and parameters but omit observe
   var prior_propensity_to_help = sample(PriorDistribution);
   var priorPredictive = binomial(prior_propensity_to_help, n);

   return {
       prior: prior_propensity_to_help, priorPredictive : priorPredictive,
       posterior : propensity_to_help, posteriorPredictive : posteriorPredictive
    };
}

var posterior = Infer({model});

viz.marginals(posterior)
~~~~

The posterior distribution depends upon: (i) the prior distribution, (ii) the assumed generative process of the data, and (iii) the observed data. 
(ii & iii are sometimes referred to collectively as the "likelihood of the observed data").

**Exercises:**

1. Try to interpret each plot, and how they relate to each other. Why are some plots densities and others bar graphs? Understanding these ideas is a key to understanding Bayesian analysis. Check your understanding by trying other data sets, varying both `k` and `n`.

2. Try different priors on `propensity_to_help`, by changing `PriorDistribution`. to `Beta({a: 10, b: 10})`, `Beta({a: 1, b: 5})` and `Beta({a: 0.1, b: 0.1})`. Use the figures produced to understand the assumptions these priors capture, and how they interact with the same data to produce posterior inferences and predictions. (If the parameter distributions aren't beautifully smooth, don't fret. Later we will see how to get better approximations for these distributions.)

3. Predictive distributions are not restricted to exactly the same experiment as the observed data, and can be used in the context of any experiment where the inferred model parameters make predictions. In the current simple binomial setting, for example, predictive distributions could be found by an experiment that is different because it has `n' != n` observations. Change the model to implement an example of this.


## Posterior prediction and model checking

The posterior predictive distribution describes what data you should expect to see, given the model you've assumed and the data you've collected so far.
If the model is a good description of the data you've collected, then the model shouldn't be surprised if you got the same data by running the experiment again.
That is, the most likely data for your model after observing your data should be the data you observed.

It's natural then to use the posterior predictive distribution to examine the descriptive adequacy of a model.
If these predictions do not match the data *already seen* (i.e., the data used to arrive at the posterior distribution over parameters), the model is descriptively inadequate.

Imagine you are piloting your task.
You have two research assistants that you send to two different research sites to collect data.
You got your first batch of data back today: For one of your research assistants, 10 out of 10 children tested performed the helping behavior. 
For the other research assitant, 0 out of 10 children tested helped.

We'll use the `editor.put()` function to save our results so we can look at the them in different code boxes.

~~~~
// "Kids who help" in 2 experiments
var k1 = 0;
var k2 = 10;

// Number of kids in 2 experiments
var n1 = 10;
var n2 = 10;

var model = function() {

  // "true effect in the population"
  var p = uniform(0, 1);

  // observed data from 2 experiments
  observe(Binomial({p: p, n: n1}), k1);
  observe(Binomial({p: p, n: n2}), k2);

  // posterior prediction
  var posteriorPredictive1 = binomial(p, n1)
  var posteriorPredictive2 = binomial(p, n2)

  return {
    parameter : p,
    predictive: {
      predictive1: posteriorPredictive1,
      predictive2: posteriorPredictive2
    }
  };
}

var opts = {
  model: model,
  method: "MCMC", callbacks: [editor.MCMCProgress()],
  samples: 20000, burn: 10000
};

var posterior = Infer(opts);

var posteriorPredictive = marginalize(posterior, "predictive")
// save results for future code boxes
editor.put("posteriorPredictive", posteriorPredictive)

var parameterPosterior = marginalize(posterior, "parameter")
viz.density(parameterPosterior, {bounds: [0, 1]})
~~~~

Looks like a reasonable posterior distribution.

How does the posterior predictive look?

~~~~
var posteriorPredictive = editor.get("posteriorPredictive")
viz(posteriorPredictive)
~~~~

This plot is a heat map because our posterior predictive distributions is over two dimensions (i.e., future data points collected by experimenter 1 and experimenter 2).
The intensity of the color represents the probability.

How well does it recreate the observed data?
Where in this 2-d grid would our observed data land?

Another way of visualizing the model-data fit is to examine a scatterplot.
Here, we will plot the "Maximum A-Posteriori" value as a point-estimate of the posterior predictive distribution.
If the data is well predicted by the posterior predictive (i.e., the model is able to accommodate the data well), it would fall along the y = x line.

~~~~
var k1 = 0, k2 = 10;
var posteriorPredictive = editor.get("posteriorPredictive")
var posteriorPredictiveMAP = posteriorPredictive.MAP().val
viz.scatter(
  [
   {model: posteriorPredictiveMAP.predictive1, data: k1},
   {model: posteriorPredictiveMAP.predictive2, data: k2}
  ]
)
~~~~

How well does the posterior predictive match the data?
What can you conclude about the parameter `p`?


## Inferences with Gaussians

[Note: These exercises are taken from reft:LW2014 Ch. 4.]


First, we want to infer the mean and standard deviation of a gaussian given some data. We set the prior over the mean to a Gaussian centered at 0, but with very large variance. We set the prior over the standard deviation to a uniform distribution over the interval [0,10].

~~~~
var data =  [1.1, 1.9, 2.3, 1.8];

var model = function(){
  var mu = gaussian({mu: 0, sigma: 30});
  var sigma = uniform({a:0, b:10});

  map(function(d){
    observe(Gaussian({mu: mu, sigma: sigma}), d)
  }, data)


  return {mu, sigma}
}

var Posterior = Infer({model, method: "MCMC", samples: 5000, burn: 2500})

viz.marginals(Posterior)
~~~~

**Exercise**: Try a few data sets, varying what you expect the mean and standard deviation to be, and how many data points you observe. 

Some suggestions:

1. Gaussian generated data (HINT: use `var data = repeat(4, function(){gaussian(5, 2)})` for 4 data points with underlying mu = 5  and sigma = 2)
2. a longer list of data (e.g., length 10 instead of length 4)
3. uniformly generated data

**Exercise**: Above we plotted the [marginal distributions](https://en.wikipedia.org/wiki/Marginal_distribution), which provides probabilities to values of a variable without regard to other variables. Try plotting the [joint distribution](https://en.wikipedia.org/wiki/Joint_probability_distribution) by simply calling `viz()` on the posterior.  Interpret the shape of the joint posterior.




### The seven scientists

Seven scientists with wildly-differing experimental skills all make a measurement of the same quantity. They get the answers x = {−27.020, 3.570, 8.191, 9.898, 9.603, 9.945, 10.056}. Intuitively, it seems clear that the first two scientists are pretty inept measurers, and that the true value of the quantity is probably just a bit below 10. The main problem is to find the posterior distribution over the measured quantity, telling us what we can infer from the measurement. A secondary problem is to infer something about the measurement skills of the seven scientists.

~~~~
var data = [-27.020, 3.570, 8.191, 9.898, 9.603, 9.945, 10.056]

var model = function() {

  var mu = gaussian(0, 30);
  var sigmas = repeat(7, function(){ uniform(0, 20) });

  mapIndexed(function(i, d){
    observe(Gaussian({mu, sigma: sigmas[i]}), d)
  }, data)

  return sigmas.concat(mu)
}

viz.marginals(Infer({model, method: "MCMC", samples: 10000,
                     callbacks: [editor.MCMCProgress()]}))
~~~~

### Repeated measures of IQ

The data are the measures $$x_{ij}$$ for the $$i = 1, . . . ,n$$ people and their $$j = 1, . . . ,m$$ repeated test scores.

We assume that the differences in repeated test scores are distributed as Gaussian error terms with zero mean and unknown precision. The mean of the Gaussian of a person’s test scores corresponds to their latent true IQ. This will be different for each person. The standard deviation of the Gaussians corresponds to the accuracy of the testing instruments in measuring the one underlying IQ value. We assume this is the same for every person, since it is conceived as a property of the tests themselves.


~~~~
var data = {
  p1: [90, 95, 100],
  p2: [105, 110, 115],
  p3: [150, 155, 160]
}

var model = function() {
  // everyone shares same sigma (corresponding to measurement error)
  var sigma = uniform(0, 50)

  // each person has a separate latent IQ
  var mus = mapObject(function(key, val){
    return uniform(0, 200)
  }, data)

  mapObject(function(key, vals){
    map(function(d){
      observe(Gaussian({mu: mus[key], sigma: sigma}), d)
    }, vals)
  }, data)

  return extend(mus,{sigma})

}

viz.marginals(Infer({model, method: "MCMC", samples: 10000}))
~~~~


In the [next chapter](04-hypothesisTesting.html), we'll see how to compare multiple hypotheses.

<!-- **TODO: This doesn't yet quite make the point about what a model check is. Show example of typical posterior predictive scatter plot?**
 -->
<!--
### Exercises 2

1.  What do you conclude about the descriptive adequacy of the model, based on the relationship between the observed data and the posterior predictive distribution? Recall the observed data is `k1 = 0; n1 = 10` and  `k2 = 10; n2 = 10`.

2. What can you conclude about the parameter `theta`?
-->

<!--
Basics from [PPAML school](http://probmods.github.io/ppaml2016/chapters/5-data.html)
-->



<!-- 
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

 -->


