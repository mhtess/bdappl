---
layout: chapter
title: Why analyze data
description: "Course overview"
---

## Learning about hypotheses from data

The goal of science is to learn about hypotheses or theories through the process of designing experiments and collecting data. 
Hypotheses are unobservable (or, latent): You cannot (in any literal sense) see Newton's equations, but they do generate *predictions* about things you can observe, such as the trajectory of an object through space. 

In addition, we are rarely in a position to make *deterministic predictions* (especially concerning social or psychological phenomena): When we hypothesize "Group A will exhibit less free will than Group B", we are making a statistical (or probabilistic) prediction. We mean: "In general, Group A will exhibit ..."
Observing one member of Group B exhibiting more free will than one member of Group A does not falsify the hypothesis. 

To learn about hypotheses from data, you should use statistics. 
There are two primary uses of statistics: For learning about aspects of single hypothesis (*parameter learning*) and for comparing two or more hypotheses against each other (*hypothesis testing*, or *model comparison*). 
It is argued that hypothesis testing is logically prior to parameter learning (e.g., you want to know that a thing exists before you go out and study its properties), but the logic and computations associated with hypothesis testing are more complex than parameter learning (reft:Wagenmakers2016:benefits; reft:LW2014). 
Thus, many Bayesian tutorials begin with parameter learning before moving onto hypothesis testing; we will follow suit here.

## Two statistics

- Probability is the mathematical foundation of statistics
- What is a probability? (When you say the probability of an event is 0.75, what does that mean?)
	- Classical definition (assumes outcomes are equally probable): Number of favorable outcomes divided by the total number of outcomes 
	- Frequentist / objectivist definition: Limiting relative frequency
	- Bayesian / subjectivist definition: Degree of belief


### Bayes Theorem follows from the definition of conditional probability

1. $$ P(B \mid A) := \frac{P(B \cap A)}{P(A)} $$
2. $$ P(A \mid B) := \frac{P(A \cap B)}{P(B)}  $$
3. $$ P(B \cap A) = P(A \cap B) $$
4. $$ P(B \mid A) \cdot P(A) = P(A \mid B) \cdot P(B) $$ 
5. $$ P(B \mid A) = \frac{P(A \mid B) \cdot P(B)}{ P(A) } $$ 

### Bayes Theorem in science


$$
P(\text{hypothesis} \mid \text{data}) = \frac{P(\text{data} \mid \text{hypothesis}) \times P(\text{hypothesis})}{P(\text{data})} $$

$$
= \frac{P(d \mid h) \times P(h)}{\int_{h} P(d \mid h)\times P(h)}
$$
 

In other words:

$$
posterior = \frac{\text{likelihood} \times \text{prior}}{\text{marginal likelihood}}
$$

Since $$posterior$$ has to be a probability, you can often get away with just saying:

$$
posterior \propto \text{likelihood} \times \text{prior}
$$

and later "normalizing" the posterior probabilities so they add to 1. 

#### A note on frequentism

- Frequentists don’t believe in $$P(h)$$
- for them: hypotheses are either true or false (doesn’t make sense to talk about the probability of a hypothesis)
- if you don’t have priors, you don’t have posteriors...  
- that’s why p-values are only about $$P(d \mid h)$$

## Why Bayesian?

- How we scientists think about statistics refp:Hoekstra2014:misinterpretation
- The inferences we would like to draw refp:Wagenmakers2016:benefits
- Tools are developed for flexibility and continuity with cognitive modeling (described below)
- **Why are we not Bayesian already?** Frequentist problems are computationally simpler


## Why use a probabilistic programming language (PPL)

A [probabilistic programming language (PPL)](https://en.wikipedia.org/wiki/Probabilistic_programming_language) is one specialized for building probabilistic models and *performing inference* in those models. 

Two reasons: (1) An abstraction barrier between the model and the algorithm for inference (a la `lm` in R) and (2) Flexibility in model development. 

Getting results out of probabilistic models requires *performing Bayesian inference* on the model. 
Probabilistic programming provides an abstraction barrier (i.e., a separation) between the theoretically-interesting *model* and the *inference algorithm* needed to return an answer from the model. 
If the probabilistic language is sufficiently well-developed, one can perform inference on a model even with only cursory knowledge of the algorithm for inference.
We will cover a few (common) algorithms in WebPPL, already part of the ecosystem.
You can use them, just by specifying some options.

Using a full-blown probabilistic *language* gives you maximal flexibility for model development and is a common language for both data analysis and cognitive modeling. 
The fundamental strokes of model development are (a) basic programming techniques; (b) a primitive way to define random variables (elementary random primitives or ERPs; a feature of probabilistic languages) and (c) a primitive way to perform inference on a model (the `Infer()` operator in WebPPL).
By employing these fundamental strokes, one can develop increasingly sophisticated models.

> **Bayesian cognitive models vs. Bayesian data analytic models**: Bayesian cognitive modeling is a research program in Cognitive Science that assumes that human cognition can be modeled as performing Bayesian inference (so-called "Bayes in the head"). Bayesian data analysis is a methodology for use in any empirical science: It assumes a particular interpretation of probability and that scientific questions (hypothesis testing, etc.) can be modeled as Bayesian inference (so-called "Bayes in the notebook")

Further, one can write model data analytic models and cognitive models in a PPL.
You can even perform a Bayesian data analysis of a Bayesian cognitive model, a so-called *descriptive Bayesian approach* refp:tauber2017descriptive.
For example, Bayesian models of cognition require specifying people's prior; one can infer what the prior knowledge should be given the observed data and the supposed model of cognition. 
If you are able to specify multiple hypotheses in a probabilistic programming language as well as a space of experiments that could potentially disambiguiate the hypotheses, one can automate the search for the optimal experiment in the same ecosystem refp:ouyangwebppl. 
<!-- Cognitive modeling thought of as a separate domain from data analysis: Everybody does data analysis, not everyone models.  -->
<!-- New perspective: AS you start building data analytic models, this leads into cognitive modeling. (At some point, you transition.) -->


### Other languages

In this tutorial, we will use the probabilistic programming language [WebPPL](http://webppl.org). There are many other good programming systems for Bayesian modeling. The two most common are:

- [STAN](http://mc-stan.org) (and now, [`brms`](https://github.com/paul-buerkner/brms))
- [BUGS](http://www.openbugs.net/w/FrontPage) / JAGS

STAN, BUGS, and JAGS are all "restricted languages". They are not full probabilistic languages and thus one cannot specify certain models in those languages. 

There are many other probabilistic languages. A new language that tries to bring the best of neural network modeling to probabilistic programming is called [Pyro](http://pyro.ai)


> Dive into the deep end with a Bayesian data analysis of a Bayesian cognitive model of the "Spinning coins" problem [here](https://probmods.org/chapters/14-bayesian-data-analysis.html).


In the [next chapter](02-introPPL.html), we will dig into the basics of probabilistic programming.



## References

cite:Hoekstra2014:misinterpretation

cite:LW2014

cite:tauber2017descriptive

cite:ouyangwebppl

cite:Wagenmakers2016:benefits


