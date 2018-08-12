---
layout: chapter
title: BDA Fundamentals
description: "The complete nuts and bolts"
---

We have now walked through the basic ideas and tools for computing a posterior distribution. We will now dive into some more details to better understand the posterior and review some of the above concepts from a different perspective. 
Much of this material is taken from the [BDA Chapter](https://probmods.org/chapters/14-bayesian-data-analysis.html) of the [probmods.org](http://probmods.org) web-book. 

Let us first rewrite our simple Bayesian data analysis model from the previous chapter, using the language of `Infer()` and `observe()`. 

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

# Comparing hypotheses

> “The Bayesian method is comparative. It compares the probabilities of the observed event on the null hypothesis and on the alternatives to it. In this respect, it is quite different from Fisher’s approach, which is absolute in the sense that it involves only a single consideration, the null hypothesis. All our uncertainty judgments should be comparative: there are no absolutes here.”  (Dennis Lindley, 1993)

In the above examples, we've had a single data-analysis model and used the experimental data to learn about the parameters of the models and the descriptive adequacy of the models.
Often as scientists, we are in the fortunate position of having multiple, distinct models in hand, and want to decide if one or another is a better description of the data.
This alternative model could be a "null model", as made popular by Null Hypothesis Significance Testing, but it need not be: It could simply be an alternative, substantive hypothesis. 
For this example, we will compare our model to a null model, where the propensity to help is fixed to be 0.5

One way of performing model comparison is to construct a "supermodel", that includes each of the hypotheses as "submodels".
In this case, the supermodel has a binary decision parameterthat we wanted to learn about (which one of the models was better).
Model comparison is then a special case of learning about the parameters of a model.
We did this by having a binary decision variable  (`flip(0.5)`) gate between which of our two models we let generate the data.
We then go backwards (performing Bayesian inference) to decide which model was more likely to have generated the data we observed.
Using this to now compare models:

~~~~
var k = 7, n = 20;

var compareModels = function() {

  // binary decision variable for which hypothesis is better
  var x = flip(0.5) ? "simple" : "complex";
  var p = (x == "simple") ? 0.5 : uniform(0, 1);

  observe(Binomial({p: p, n: n}), k);

  return {model: x}
}

var opts = {model: compareModels, method: "rejection", samples: 2000};
print("We observed " + k + " successes out of " + n + " attempts")
var modelPosterior = Infer(opts);
viz(modelPosterior)
~~~~

This supermodel is an example of Bayesian Null Hypothesis Testing.
We consider a model that fixes one of its parameters to a pre-specified value of interest (here $$\mathcal{H_0} : p = 0.5$$).
This is sometimes referred to as a *null hypothesis*.
The other model says that the parameter is free to vary.
In the classical hypothesis testing framework, we would write: $${H_1} : p \neq 0.5$$.
With Bayesian hypothesis testing, we must be explicit about what $$p$$ is (not just what p is not), so we write $${H_1} : p \sim \text{Uniform}(0, 1) $$.

One might have a conceptual worry: Isn't the second model just a more general case of the first model?
That is, if the second model has a uniform distribution over `p`, then `p: 0.5` is included in the second model.
This is what's called a *nested model*.

Shouldn't the more general model always be better?
If we're at a track, and you bet on horse A, and I bet on horse A and B, aren't I strictly in a better position than you?
The answer is no, and the reason has to do with our metric for winning.
Intuitively, we don't care whether your horse won or not, but how much money you win.
How much money you win depends on how much money you bet, and the rule is, when we go to track, we have the same amount of money.

In probabilistic models, our money is probabilities. Each model must allocate its probability so that it sums to 1.
So my act of betting on horse A and horse B actually requires me to split my money (say, betting 50 / 50 on each).
On the other hand, you put all your money on horse A (100 on A, 0 on B).
If A wins, you will gain more money because you put more money down.

This idea is called the principle of parsimony or Occam's razor.
More complex models will be penalized for being more complex, intuitively because they will be diluting their predictions.
At the same time, more complex models are more flexible and can capture a wider variety of data (they are able to bet on more horses, which increases the chance that they will win some money).
Bayesian model comparison lets us weigh these costs and benefits.

## Bayes' factor

What we are plotting above are **posterior model probabilities**.
These are a function of the marginal likelihoods of the data under each hypothesis and the prior model probabilities (here, defined to be equal: `flip(0.5)`).
Sometimes, scientists feel a bit strange about reporting values that are based on prior model probabilities (what if scientists have different priors as to the relative plausibility of the hypotheses?) and so often report the ratio of marginal likelihoods, a quantity known as a *Bayes Factor*.

> Marginal likelihood: The likelihood of the data averaged (marginalized) over the prior distribution over the parameters: $$\int_\theta P(d \mid \theta) \cdot P(\theta)$$. It is the average quality of predictions of the hypothesis for the observed data. In order for the marginal likelihood to be high, hypotheses needs to make a lot of good predictions. “A hypothesis that predicts everything predicts nothing”


> Bayes Factor: The ratio of the marginal likelihoods for two hypotheses. $$BF_{10} = \frac{P(d \mid \mathcal{H_1})}{P(d \mid \mathcal{H_0})}$$. The Bayes Factor tells you how much better one hypothesis is at predicting the observed data than the other, taking into account the flexibility of each hypothesis.

Bayes Factors have a continuous interpretation. Still, category judgments are sometimes useful. Check out the wikipedia entry on Bayes Factors for the common ways to [interpret Bayes Factors](https://en.wikipedia.org/wiki/Bayes_factor#Interpretation). 


Let's compute the Bayes' Factor, by computing the likelihood of the data under each hypothesis.

~~~~
var k = 7, n = 20;

var simpleLikelihood = Math.exp(Binomial({p: 0.5, n: n}).score(k))

var complexModel = Infer({
  model: function(){
    var p = uniform(0, 1);
    return binomial(p, n)
  }, 
  method: "forward", samples: 10000
})

var complexLikelihood = Math.exp(complexModel.score(k))

var bayesFactor_01 = simpleLikelihood / complexLikelihood
bayesFactor_01
~~~~


How does the Bayes Factor in this case relate to posterior model probabilities above?

Here, we have estimated the likelihood of the data under the simple model using ["forward sampling"](https://webppl.readthedocs.io/en/master/inference/methods.html#forward-sampling), a simple inference algorithm that just repeatedly samples. 
Often in practice this will be tremendously inefficient (e.g., when your observed data `k` is really unlikely, which is easy to occur when you have a data set that is sufficiently large). 
Current research in computer science is trying to develop better algorithms for approximating the marginal likelihood.
One good technique is called [Annealed Importance Sampling](https://arxiv.org/abs/physics/9803008), and will be available in WebPPL very soon. 

## Savage-Dickey method

For this example, the Bayes factor can be obtained by integrating out the model parameter (using `Infer` with `{method: "forward"}`).
However, it is not always easy to get good estimates of the two marginal probabilities.
It turns out, the Bayes factor can also be obtained by considering *only* the more complex hypothesis ($$\mathcal{H}_1$$).
What you do is look at the distribution over the parameter of interest (here, $$p$$) at the point of interest (here, $$p = 0.5$$).
Dividing the probability density of the posterior by the density of the prior (of the parameter at the point of interest) also gives you the Bayes Factor!
This perhaps surprising result was described by Dickey and Lientz (1970), and they attribute it to Leonard "Jimmie" Savage.
The method is called the *Savage-Dickey density ratio* and is widely used in experimental science.

We would use it like so:

~~~~
var k = 7, n = 20;

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
print( savageDickeyRatio )
~~~~

(Note that we have approximated the densities using a [kernal density estimator](https://webppl.readthedocs.io/en/master/distributions.html#KDE). Another way to estimate the density is by looking at the expectation that $$p$$ is within $$0.05$$ of the target value $$p=0.5$$.)

For more information about different methods of calculating Bayes Factors, see [this helpful blogpost](http://michael-franke.github.io/statistics,/modeling/2017/07/07/BF_computation.html).

We have now gone through the fundamentals of Bayesian Data Analysis.
In the [next chapter](5-advancedBDA.html), we'll show how you can elaborate your models to better represent your data.
