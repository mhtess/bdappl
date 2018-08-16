---
layout: chapter
title: Comparing hypotheses
description: "Hypothesis testing is model comparison"
---

> “The Bayesian method is comparative. It compares the probabilities of the observed event on the null hypothesis and on the alternatives to it. In this respect, it is quite different from Fisher’s approach, which is absolute in the sense that it involves only a single consideration, the null hypothesis. All our uncertainty judgments should be comparative: there are no absolutes here.”  (Dennis Lindley, 1993)

In the examples so far, we've had a single data-analysis model and used the experimental data to learn about the latent parameters of the models and the descriptive adequacy of the models.
Often as scientists, we are in the (fortunate) position of having multiple, distinct models in hand, and want to decide if one or another is a better description of the data.

Consider a case where we have two hypotheses/models we wants to compare: $$\mathcal{M}_0$$ and $$\mathcal{M}_1$$
If we want to be take the Bayesian to model comparison, we first specify our prior beliefs about each of the hypothesis: $$P(\mathcal{M}_0)$$ and $$P(\mathcal{M}_1)$$.
Then, our *posterior belief* concerning the probability of $$\mathcal{M}_{0}$$ after observing data $$D$$ can be calculated by Bayes rule:

$$ P(\mathcal{M}_{0} \mid D) = \frac{ P(D \mid \mathcal{M}_{0}) \cdot P(\mathcal{M}_{0}) }{P(D \mid \mathcal{M}_{0}) \cdot P(\mathcal{M}_{0})   +  P(D \mid \mathcal{M}_{1}) \cdot P(\mathcal{M}_{1}) }$$

Schematically in WebPPL, this is just a straight-forward inference model, where the variable of interest ("which model is correct?") is a bernoulli random variable (a flip) and we infer the most likely hypothesis/model given the data.
<!-- The most general Bayesian solution to this problem is to simply perform Bayesian inference over infer the most likely hypothesis given the data.  -->
<!-- One fully Bayesian way of doing model comparison is to simply elaborate the probabilistic program to represent the uncertainty about which is the better hypothesis. -->


~~~~ norun
var model_comparison_model = function(){
  var m = flip() ? Model_1 : Model_0
  observe(m, d)
  return m
}
~~~~

This way of casting the model comparison problem is sometimes called a "supermodel", which includes each of the hypotheses as "submodels".

Let's see an example: Consider our children helping study from the previous study. 
Rather than estimate the `propensity_to_help`, we might want to perform a "Null Hypothesis" test: What is the probability that the behavior observed in our experiment (that 15 out of 20 kids helped) could be explained by true randomness (i.e., `propensity_to_help = 0.5`)?
Note that this question ($$P(\mathcal{M}_0 \mid D)$$) can only be addressed by Bayesian methods.
Frequentist methods (e.g., p-values) do not ascribe probabilities to hypotheses; a p-value is a description of the probability of the *data* *assuming* the null hypothesis is true: $$P(D \mid \mathcal{M}_0)$$. (A p-value is actually more complicated than this: It is the probability of the observed data, *comparable data*, or *more exteme data* assuming the null hypothesis is true. If you're not sure what exactly that means, then you are thinking about it correctly. p-values are famously unintuitive.)

Bayesian Null Hypothesis Testing requires you to specify *both* hypotheses: It is not enough to define what the null hypothesis is.
The alternative hypothesis is intuitively $$\mathcal{M}_1: \theta \neq 0.5$$, but we must be more specific. 
One standard way of defining the null hypothesis is to put say that the parameter is drawn from a uniform distribution.(Note that this was the model from the previous chapter.)


~~~~
var k = 15, n = 20; // number of observed successes, total number of trials

var Null_model = Delta({v: 0.5}) // null hypothesis is a Delta distribution at 0.5
var Alternative_model = Uniform({a: 0, b: 1})

var super_model = function() {

  // binary decision variable for which hypothesis is better
  var the_better_model = uniformDraw(["null", "alternative"])

  var p = sample(the_better_model == "null" ? Null_model : Alternative_model)

  observe(Binomial({p: p, n: n}), k);

  return {the_better_model}
}

var opts = {model: super_model, method: "rejection", samples: 2000};
print("We observed " + k + " successes out of " + n + " attempts")
var modelPosterior = Infer(opts);
viz(modelPosterior)
~~~~



<!-- One fully Bayesian way of doing model comparison is to simply elaborate the probabilistic program to represent the uncertainty about which is the better hypothesis. -->



<!-- Since $$P(\mathcal{M}_0) = P(\mathcal{M}_1)$$, this then reduces to: 
 -->
<!-- $$ P(\mathcal{M}_{0} \mid D) = \frac{ P(D \mid \mathcal{M}_{0})  }{P(D \mid \mathcal{M}_{0})  +  P(D \mid \mathcal{M}_{1}) }$$
 -->


<!-- Since there are only two hypotheses, we know that $$P(\mathcal{M}_0) + P(\mathcal{M}_1) = 1$$.
A standard assumption would be to stipulate these prior beliefs be equal (no preference for either hypothesis/model *a priori*): $$P(\mathcal{M}_0) = P(\mathcal{M}_1) = 0.5$$.
 -->


<!-- $$ P(\text{hypothesis} \mid \text{data}) \propto \frac{P(\text{data} \mid \text{hypothesis}) \times P(\text{hypothesis})}{P(\text{data})} $$ -->



<!-- 
Here, we are implementing model comparison as a special case of learning about the parameters of a model (the "super_model"). 
The parameter of interest is `the_better_model`.

This supermodel is an example of Bayesian Null Hypothesis Testing.
We consider a model that fixes one of its parameters to a pre-specified value of interest (here $$\mathcal{H_0} : p = 0.5$$).
This is sometimes referred to as a *null hypothesis*.
The other model says that the parameter is free to vary.
In the classical hypothesis testing framework, we would write: $${H_1} : p \neq 0.5$$.
With Bayesian hypothesis testing, we must be explicit about what $$p$$ is (not just what p is not), so we write $${H_1} : p \sim \text{Uniform}(0, 1) $$.
 -->
One might have a conceptual worry: Isn't the second model just a more general case of the first model?
That is, if the second model has a uniform distribution over `p`, then `p: 0.5` is included in the second model.
This is what's called a *nested model*.

Shouldn't the more general model always be better?
If Bart and Lisa are at a track, and Lisa bets on horse A, and Bart bets on horse A and B, isn't Bart strictly in a better position than Lisa?
Well, no.
Gambling is not about whose horse won or not, but how much money you win.
If Bart and Lisa each have $100: Lisa puts all of her money on Horse A, while Bart splits his money between Horse A and Horse B (betting $50 on each). 
Then if Horse A wins, Bart and Lisa will both win money but Lisa will more money.
In probabilistic models, the currency is probability. 
Each model can allocate its probability over different parameters value but those probabilities must add to 1.

This idea is called the principle of parsimony or Occam's razor.
More complex models will be penalized for being more complex, intuitively because they will be diluting their predictions.
At the same time, more complex models are more flexible and can capture a wider variety of data (they are able to bet on more horses, which increases the chance that they will win some money).
Bayesian model comparison lets us weigh these costs and benefits.


**Exercises**

1. Under a frequentist binomial test, the p-value for 15 successes out of 20 trials is $$p = 0.0414$$. What does the Bayesian model comparison conclude? 

2. Is the Bayesian or the Frequentist more conservative in this case? Why do you think that might be?

3. Imagine we run another experiment where we collect 36 participants and 20 of them helped (`k = 20, n = 36`). What is the p-value under a binomial test? (You can use R, or some online gui.) What could you conclude from the p-value? What are the posterior model probabilities? What can you conclude from them?




## Bayes' factor

So far we have considered the **posterior model probabilities** as a way to quantitatively compare models.
These are a function of the likelihoods of the data under each hypothesis and the prior model probabilities (above defined to be equal: `flip(0.5)`).
Sometimes, scientists feel a bit strange about reporting values that are based on prior model probabilities (what if scientists have different priors as to the relative plausibility of the hypotheses?) and so often report the ratio of likelihoods, a quantity known as a *Bayes Factor*.

Often, the likelihood $$P(D | \mathcal{M})$$ is not just a single value because model $$\mathcal{M}$$ will have some latent parameter(s) $$\theta$$ that are not specified *a priori*. 
For example, under $$\mathcal{M}_1$$, the `propensity_to_help` variable (which we will call $$\theta$$ for simplicity) is drawn from uniform distribution. 
Thus, to compute $$P(D | \mathcal{M})$$, you must first consult $$P(D | \mathcal{M}, \theta)$$.
For some values of $$\theta$$, $$P(D | \mathcal{M}, \theta)$$ will be high (e.g., if $$D$$ is 15 out of 20 heads, then $$P(D | \mathcal{M}, \theta)$$ will be highest when $$\theta = 0.75$$).
For other values of $$\theta$$, $$P(D | \mathcal{M}, \theta)$$ will be low (e.g., $$\theta = 0.25$$ for 15 ouf of 20 heads). 
$$P(D | \mathcal{M})$$ is the likelihood of the data $$D$$ under model $$\mathcal{M}$$ averaged over the parameters $$\theta$$. This is called the **marginal likelihood**.

> Marginal likelihood: The likelihood of the data averaged (marginalized) over the prior distribution over the parameters: $$\int_\theta P(D \mid \theta) \cdot P(\theta)$$. It is the average quality of predictions of the hypothesis for the observed data. In order for the marginal likelihood to be high, hypotheses needs to make a lot of good predictions. “A hypothesis that predicts everything predicts nothing”

The marginal likelihood is not itself directly interpretable but is interpretable in a comparative way.
Hence, scientists often report the ratio of marginal likelihoods for two models, called a **Bayes Factor**.


> Bayes Factor: The ratio of the marginal likelihoods for two hypotheses. $$BF_{10} = \frac{P(D \mid \mathcal{M_1})}{P(D \mid \mathcal{M_0})}$$. The Bayes Factor tells you how much better one hypothesis is at predicting the observed data than the other, taking into account the flexibility of each hypothesis.

Bayes Factors have a continuous interpretation. Still, category judgments are sometimes useful. Check out the wikipedia entry on Bayes Factors for the common ways to [interpret Bayes Factors](https://en.wikipedia.org/wiki/Bayes_factor#Interpretation). 

In order to compute a Bayes Factor, you just need to compute the marginal likelihoods for each model.
Unfortunately, this is computationally challenging in the general case.
(In general, estimating the integral over $$\theta$$ is difficult).
Here, we'll discuss two ways of estimating the marginal likelihood, and then return to a special case of computing Bayes Factors which side-steps the issue of estimating marginal likelihoods.
For a more in-depth discussion of several methods for calculating Bayes Factors, see [this helpful blogpost](http://michael-franke.github.io/statistics,/modeling/2017/07/07/BF_computation.html).

### Estimation of Marginal Likelihood by forward sampling

The most naive way of estimating marginal likelihoods is by "forward sampling".
Recall, we want to compute the likelihood of the data averaged over the prior on parameters.


~~~~
var k = 15, n = 20;

var complexModel = function(){
  var p = uniform(0, 1);
  return Math.exp(Binomial({p, n}).score(k)) // p(d | theta)
}

// approximate the integral by sampling
var likelihood_Dist = repeat(10000, complexModel)
var complexLikelihood = sum(likelihood_Dist) / likelihood_Dist.length

// simple model doesn't have any parameters to integrate over
var simpleLikelihood = Math.exp(Binomial({p: 0.5, n: n}).score(k))

var bayesFactor_10 = complexLikelihood / simpleLikelihood
bayesFactor_10
~~~~

The technique of repeatedly sampling is very common and has it's own inference method in WebPPL called ["forward"](https://webppl.readthedocs.io/en/master/inference/methods.html#forward-sampling).

~~~~
var k = 15, n = 20;

var simpleLikelihood = Math.exp(Binomial({p: 0.5, n: n}).score(k))

var complexModel = Infer({
  model: function(){
    var p = uniform(0, 1);
    return binomial(p, n)
  }, 
  method: "forward", samples: 10000
})

var complexLikelihood = Math.exp(complexModel.score(k))

var bayesFactor_10 = complexLikelihood /  simpleLikelihood
bayesFactor_10
~~~~

**Exercise**: How does the Bayes Factor in this case compare to posterior model probabilities above?


### Estimation of Marginal Likelihood by Annealed Importance Sampling

Forward sampling, though straight-forward to understand, is in practice tremendously inefficient (e.g., when your observed data `k` is really unlikely, which is easy to occur when you have a data set that is sufficiently large). 
Current research in computer science is trying to develop better algorithms for approximating the marginal likelihood.
One good technique is called [Annealed Importance Sampling](https://arxiv.org/abs/physics/9803008).


[AIS docs](https://webppl.readthedocs.io/en/master/functions/other.html?highlight=ais#AIS)

~~~~
var k = 15, n = 20;

var complexModel = function(){
  var p = uniform(0, 1);
  observe(Binomial({p, n}), k)
}

var simpleModel = function(){
  observe(Binomial({p: 0.5 , n}), k)
}

// increasing steps decreases the variance in the estimate
// samples is the number of repetitions of the procedure (result is averaging over samples)
var log_likelihood_m1 = AIS(complexModel, {steps: 10000, samples: 5})

// overkill: can just compute this via Binomial.score(k)
var log_likelihood_m0 = AIS(simpleModel)

var log_bf_10 = log_likelihood_m1 - log_likelihood_m0
Math.exp(log_bf_10) // BF_10
~~~~


## Savage-Dickey method

Above, we obatined the Bayes factor by integrating out the model parameter (by way of various methods).
As we have said, it is not always easy to get good estimates of the two marginal likelihoods.
It turns out, the Bayes factor can also be obtained by considering *only* the more complex hypothesis ($$\mathcal{M}_1$$).
What you do is look at the distribution over the parameter of interest (here, $$p$$) at the point of interest (here, $$p = 0.5$$).
Dividing the probability density of the posterior by the density of the prior (of the parameter at the point of interest) also gives you the Bayes Factor!
This perhaps surprising result was described by Dickey and Lientz (1970), and they attribute it to Leonard "Jimmie" Savage.
The method is called the *Savage-Dickey density ratio* and is widely used in experimental science.

We would use it like so:

~~~~
var k = 15, n = 20;

var complexModelPrior = Infer({method: "forward", samples: 10000}, function(){
  var p = uniform(0, 1);
  return p
})

var complexModelPosterior = Infer({method: "rejection", samples: 10000}, function(){
  var p = uniform(0, 1);
  observe(Binomial({p: p, n: n}), k);
  return p
})

// approximate density by looking at expectation with range of 0.05
// var savageDickeyDenomenator = expectation(complexModelPrior, function(x){return Math.abs(x-0.5)<0.05})
// var savageDickeyNumerator = expectation(complexModelPosterior, function(x){return Math.abs(x-0.5)<0.05})

// approximate density using built-in kernal density estimator
var savageDickeyDenomenator = Math.exp(kde(complexModelPrior).score(0.5))
var savageDickeyNumerator = Math.exp(kde(complexModelPosterior).score(0.5))

var savageDickeyRatio = savageDickeyNumerator / savageDickeyDenomenator
savageDickeyRatio
~~~~

(Note that we have approximated the densities using a [kernal density estimator](https://webppl.readthedocs.io/en/master/distributions.html#KDE). Another way to estimate the density is by looking at the expectation that $$p$$ is within $$0.05$$ of the target value $$p=0.5$$.)



### The research assistants

Recall our example from the previous chapter about two research assistants who returned wildly different results. 
We had the intuition that they were doing something different from one another.
Is there a way to quantify our degree of belief in that proposition?

We can take a model comparison approach, comparing a model where we posit a single latent parameter (research assistants are doing / measuring the same thing) vs. a model where we posit different parameters for the different research assistants. 
The latter model has more flexibility, but it also probably predicts the observed data better.

~~~~
// "Kids who help" in 2 experiments
var k1 = 0;
var k2 = 10;

// Number of kids in 2 experiments
var n1 = 10;
var n2 = 10;

var simple_model = function(){
  var p1 = uniform(0,1);
  observe(Binomial({p: p1, n: n1}), k1);
  var p2 = p1;
  observe(Binomial({p: p2, n: n2}), k2);
}

var complex_model = function(){
  var p1 = uniform(0,1);
  observe(Binomial({p: p1, n: n1}), k1);
  var p2 = uniform(0, 1)
  observe(Binomial({p: p2, n: n2}), k2);
}

var ll1 = AIS(simple_model, {steps: 10000, samples: 10})
var ll2 = AIS(complex_model, {steps: 10000, samples: 10})

Math.exp(ll2 - ll1)
~~~~


We have now gone through the fundamentals of Bayesian Data Analysis.
In the [next chapter](05-patterns.html), we'll show you basic techniques for modeling causality among random variables. 
