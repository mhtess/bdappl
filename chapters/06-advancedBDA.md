---
layout: chapter
title: Elaborating models
description: "A pinch of sophistication and elegance"
custom_js:
- /assets/js/exptData.js
- /assets/js/bdaHelpers.js
---

Imagine you are running a study to examine the efficacy of social messaging on participants' belief in a cause (e.g., eating less meat).
In your study, participants read about the behavior of others in their commmunity and report on their interest in eating less meat (e.g., as in [this paper](https://pdfs.semanticscholar.org/d172/6f274e7c0f7190f83854352d8ffcbda632bb.pdf)). 
Later in the study, they are given the opportunity to donate their monetary compensation to a non-profit that supports the cause. 

## Beginning with a simple model

You run your study on facebook and collect 1000 participants worth of data.
Your data set includes the messaging condition (experimental vs. control), the amount of time they spent on the experiment, their rating of their belief in the cause, whether or not the participant donated their earnings, and miscellaneous demographic data (their browser, location).

Let's take a look at the data.

~~~~
// print first few lines of data set
display(exptData.slice(0, 6))

// show all the ids
display(_.map(exptData, "id"))

// how many people saw each banner?
viz.table(_.map(exptData, "condition"))
~~~~

Your main hypothesis is that the donation rate is going to be higher for the group in the experimental condition than in the control condition. 

~~~~
var model = function() {

  // unknown rates of donation
  var donationRates = {
    experimental: uniform(0,1),
    control: uniform(0,1)
  };

  map(function(personData) {

    // grab appropriate donationRates by condition
    var acceptanceRate = donationRates[personData["condition"]];
    
    // participants are independent and identically distributed
    observe(Bernoulli({p:acceptanceRate}), personData["converted"])

  }, exptData)

  return extend(donationRates,{
    delta: donationRates.experimental - donationRates.control
  })

}

var numSamples = 10000

var inferOpts = {
  model: model,
  method: "MCMC", 
  samples: numSamples,
  burn: numSamples/2, 
  callbacks: [editor.MCMCProgress()] 
};

var posterior = Infer(inferOpts);

viz.marginals(posterior)	
~~~~

**Exercise**: By examining the figure, what can you infer about the difference between the experimental and control conditions?

## Data Contamination

You show this analysis to your colleague. She raises the concern that some participants in your data set may not be internalizing the social messaging information. Perhaps they are just clicking through the experiment. 

Fortunately, you have recorded how much time participants spend on the experimenet. Let's visualize that data.

~~~~
viz.hist(_.map(exptData, "time"), {numBins: 10})
~~~~

This look likes canonical wait time data, following a log-normal distribution. To validate this intuition, let's look at the data by taking the log.

~~~~
var logTimeData = map(function(t){
  return Math.log(t);
}, _.map(exptData, "time"))

viz.hist(logTimeData, {numBins: 10})
~~~~

Looks pretty normal, but also looks like there's something funny going on. Some people are spending substantially less time on your experiment than other people. Your colleague might be right: Presumably, none of these people clicking through the experiment are going to be internalizing the subtle social messaging. 

How can we account for this potential data contamination? We posit 2 groups of visitors: `"bonafide"` participants and `"disingenuous"` participants, and they plausibly have different rates of donating.

~~~~
///fold:
var foreach = function(lst, fn) {
    var foreach_ = function(i) {
        if (i < lst.length) {
            fn(lst[i]);
            foreach_(i + 1);
        }
    };
    foreach_(0);
};
///

var model = function() {

  // average time spent on the website (in log-seconds)
  // assume the disingenuous participants spend less time on the experiment 
  // (because they click through)
  var logTimes = {
    bonafide: gaussian(5,2), // exp(3) ~ 2m
    disingenuous: gaussian(2,2), // exp(2) ~ 7s
  }

  // variance of time spent on website (plausibly different for the two groups)
  var sigmas =  {
    bonafide: uniform(0,3),
    disingenuous: uniform(0,3),
  }

  var donationRates = {
    experimental: uniform(0,1),
    control: uniform(0,1)
  };

  // mixture parameter (i.e., % of bonafide visitors)
  var probBonafide = uniform(0,1);

  foreach(exptData, function(personData) {

    var group = flip(probBonafide) ? "bonafide" : "disingenuous";

    observe(
      Gaussian({mu: logTimes[group], sigma: sigmas[group]}), 
      Math.log(personData.time)
    )

    // disingenuous visitors have a very low probability of donating
    var acceptanceRate = (group == "bonafide") ? 
        donationRates[personData.condition] : 
    0.0000001

    observe(Bernoulli({p:acceptanceRate}), personData.converted)

  })

  return { 
    logTimes_disingenuous: logTimes.disingenuous,
    logTimes_bonafide: logTimes.bonafide,
    sigma_disingenuous: sigmas.disingenuous,
    sigma_bonafide: sigmas.bonafide,
    experimental: donationRates.experimental,
    control: donationRates.control,
    percent_bonafide: probBonafide 
  }

}

var numSamples = 100000;
var posterior = Infer({
  method: "incrementalMH", 
  samples: numSamples, burn: numSamples/2,
  verbose: true, 
  verboseLag: numSamples/10,
  model: model
})

// run a big model: takes about 1 minute
editor.put("posterior", posterior)
~~~~

Above, we are using the inference algorithm `incrementalMH`, which is efficient to use when you have many random variables (i.e., sample statements) that only impact a small set of computations. Above, the `group` variable is sampled for every participant (so there are 200 `group` variables) but each one only directly influences the likelihood of that participant's data. (This is easy to note because that variable only exists when that participant's data is being considered.)


<!-- More efficient way to write this model:

~~~~
///fold:
var foreach = function(lst, fn) {
    var foreach_ = function(i) {
        if (i < lst.length) {
            fn(lst[i]);
            foreach_(i + 1);
        }
    };
    foreach_(0);
};
///

var model = function() {

  // average time spent on the website (in log-seconds)
  // assume the disingenuous participants spend less time on the experiment (because they click through)
  var logTimes = {
    bonafide: gaussian(3,3), // exp(3) ~ 20s
    disingenuous: gaussian(0,2), // exp(2) ~ 7s
  }

  // variance of time spent on website (plausibly different for the two groups)
  var sigmas =  {
    bonafide: uniform(0,3),
    disingenuous: uniform(0,3),
  }

  var donationRates = {
    experimental: uniform(0,1),
    control: uniform(0,1)
  };

  // mixture parameter (i.e., % of bonafide visitors)
  var probBonafide = uniform(0,1);

  foreach(exptData, function(personData) {

    var logTime = Math.log(personData.time)

    factor(
        Math.log(
          probBonafide* Math.exp(Gaussian({mu: logTimes.bonafide, sigma: sigmas.bonafide}).score(logTime)) + 
          (1-probBonafide)* Math.exp(Gaussian({mu: logTimes.disingenuous, sigma: sigmas.disingenuous}).score(logTime))
          )
      )

    factor(
        Math.log(
          probBonafide* Math.exp(Bernoulli({p:donationRates[personData.condition]}).score(personData.converted)) + 
          (1-probBonafide)* Math.exp(Bernoulli({p:0.0001}).score(personData.converted))
          )
      )


  } )

  return { logTimes_disingenuous: logTimes.disingenuous,
            logTimes_bonafide: logTimes.bonafide,
            sigma_disingenuous: sigmas.disingenuous,
            sigma_bonafide: sigmas.bonafide,
            experimental: donationRates.experimental,
            control: donationRates.control,
            percent_bonafide: probBonafide }

}

var numSamples = 10000;
var posterior = Infer({  method: "MCMC", 
  samples: numSamples,
  burn: numSamples/2, 
  callbacks: [editor.MCMCProgress()] }, 
                      model)

// run a big model: takes about 1 minute
editor.put("posterior", posterior)
~~~~
 -->
#### Examine posterior

Display marginal posterior over the rate of bonafide participants.

~~~~
var jointPosterior = editor.get("posterior");

var marginalBonafide = marginalize(jointPosterior, "percent_bonafide");

viz.hist(marginalBonafide, {numBins: 15});
~~~~
	
So, indeed, almost 30% of your participants were inferred to be "disingenuous".

~~~~
///fold:
var marginalizeExponentiate = function(myDist, label){
    Infer({method: "enumerate"}, function(){
        var x = sample(myDist);
        return Math.exp(x[label])
    });
};
///

var jointPosterior = editor.get("posterior");

var marginalTime_disingenuous = marginalizeExponentiate(jointPosterior, "logTimes_disingenuous");
var marginalTime_bonafide = marginalizeExponentiate(jointPosterior, "logTimes_bonafide");

print("Inferred time spent by accidental visitors (in seconds)")
viz.hist(marginalTime_disingenuous, {numBins: 10});

print("Inferred time spent by bonafide visitors (in seconds)")
viz.hist(marginalTime_bonafide, {numBins: 10})
~~~~

**Exercises**

1. Return the marginal distributions over the rates of the donation parameters. (Use the [`marginalize()`](http://docs.webppl.org/en/master/functions/other.html#marginalize) function). Does accounting for the disingenuous participants change the conclusions you can draw about the efficacy of the experimental condition?

2. You show these results to your colleague, and she is surprised by them. Why are the results the way they are? In the above model, we assumed the rate of bonafide participants was independent of which condition they were assigned. Could that be incorrect? Modify the model above to test the hypothesis that the rate of bonafide participants was different for the different experimental conditions.

3. In the above model, we assume that disingenuous participants are very unlikely to donate. How could we relax this assumption, and say that disingenuous participants also have some probability of "accidentally" donating their money? Modify the model to express this possibility, run the model, and draw inferences about the rate at which disingenuous participants donate their earnings.

## Different, or multiple, dependent measures

So far, we have been examining the hypothesis of whether the an experimental social messaging influences participants' propensity to donate their compensation to a social cause. Your colleague reminds you that you also collected ratings about their belief in the importance of the cause. Those ratings were given on a 3-pt scale, where 2 was a strong endorsement and 0 was no endorsement of the cause.

We could analyze these data separately, and see if there was any effect on the endorsement data. But insofar as we believe these two measurements are the byproduct of the same latent construct (e.g., the underlying support for the cause), we should be able to articulate a way in which they are related.

Let's suppose that a person's support for the cause is a number between zero and two. 
Then the link to whether or not the person donates could simply be a coin flip as before (weighted by their suppor divided by 2). 
How do we handle their endorsement rating of the cause?
The measurement is a 3-pt Likert scale, which is an ordinal measurement. 
Ordinal measures are ones in which you have an ordering (e.g., r_0 < r_1 < r_2, where $$r_i$$ denotes a rating on the $$i$$th element of the scale) but you don't know the distance between the points, which is the appropriate way to interpret Likert scale data. 
Following reft:kruschke2014doing (Ch. 23), we can imagine a thresholded cumulative-normal model: a Gaussian distribution with certain cut-offs or thresholds, below or above which participants provide certain ratings on the discretized Likert scale.
Below the lowest threshold, a person provides the lowest ratings; in between thresholds, a person provides an intermediate threshold; and above the last threshold, a person provides the maximal ratings.
This model is underdetermined and so one must fix two of the thresholds; for a 3-pt scale, those are all the thresholds needed (see reft:kruschke2014doing Ch. 23 for a deeper discussion of this model). 
Our `OrdinalGaussian` function below takes three arguments: the thresholds and the mean and variance of the underlying Gaussian.
Let's assume the mean is the participant's support for the cause, and the variance is a nuisance parameter that we will try to estimate from the data.


~~~~
///fold:
var foreach = function(lst, fn) {
  var foreach_ = function(i) {
    if (i < lst.length) {
      fn(lst[i]);
      foreach_(i + 1);
    }
  };
  foreach_(0);
};

var levels = function(df, label){
  return _.uniq(_.map(df, label));
}

var erf = function(x1) {
  // save the sign of x
  var sign = (x1 >= 0) ? 1 : -1;
  var   x = Math.abs(x1);

  // constants
  var a1 =  0.254829592;
  var a2 = -0.284496736;
  var a3 =  1.421413741;
  var a4 = -1.453152027;
  var a5 =  1.061405429;
  var p  =  0.3275911;

  // A&S formula 7.1.26
  var t = 1.0/(1.0 + p*x);
  var y = 1.0 - (((((a5 * t + a4) * t) + a3) * t + a2) * t + a1) * t * Math.exp(-x * x);
  return sign * y; // erf(-x) = -erf(x);
}

var gaussianCDF = function(params){
  var x = params.x;
  var mu = params.mu ? params.mu : 0;
  var sigma =params.sigma ? params.sigma : 1;
  var erfNum = x - mu;
  var erfDen = sigma * Math.sqrt(2);
  return (1/2)*(1 + erf(erfNum/erfDen))
}

// bounds of likert scale
var lowerBound = 0, upperBound = 2, binWidth = 1;
///

// adapted from Krushcke (2014)
var OrdinalGaussian = function(params){
  var thresholds = params.thresholds;
  var mu = params.mu, sigma = params.sigma;
  var bins = _.range(lowerBound, upperBound+1, binWidth);

  var probs = mapIndexed(function(i, b){
    i == 0 ? gaussianCDF({x:thresholds[i], mu: mu, sigma: sigma}) :
    i == thresholds.length ? 1 - gaussianCDF({x:thresholds[i-1], mu: mu, sigma: sigma}) :
    Math.max(Number.EPSILON, gaussianCDF({x:thresholds[i], mu: mu, sigma: sigma}) - 
             gaussianCDF({x:thresholds[i-1], mu: mu, sigma: sigma}))
  }, bins)

  return Categorical({ps: probs, vs: bins});
}

var model = function() {

  // mixture parameter (i.e., % of bonafide visitors)
  var probBonafide = uniform(0,1);

  // average time spent on the website (in log-seconds)
  // assume the disingenuous participants spend less time on the experiment 
  // (because they click through)
  var logTimes = {
    bonafide: gaussian(5,2), // exp(3) ~ 2m
    disingenuous: gaussian(2,2), // exp(2) ~ 7s
  }

  // variance of time spent on website (plausibly different for the two groups)
  var sigmas =  {
    bonafide: uniform(0,3),
    disingenuous: uniform(0,3),
  }

  var likertSigma = uniform(0,2)

  var supportForTheCause = {
    experimental: uniform(0, 2),
    control: uniform(0, 2)
  };

  var donationRates = {
    experimental: supportForTheCause.experimental / 2,
    control: supportForTheCause.control / 2
  };


  foreach(exptData, function(personData) {

    var group = flip(probBonafide) ? "bonafide" : "disingenuous";

    observe(
      Gaussian({mu: logTimes[group], sigma: sigmas[group]}), 
      Math.log(personData.time)
    )

    // disingenuous visitors have a very low probability of donating
    var acceptanceRate = (group == "bonafide") ? 
        donationRates[personData.condition] : 
    0.0000001

    observe(Bernoulli({p:acceptanceRate}), personData.converted)

    // endorsement ratings 
    observe(
      OrdinalGaussian({
        mu: supportForTheCause[personData.condition],
        sigma: likertSigma,
        thresholds: [0.5, 1.5]
      }), personData.endorsement)

  })

  return { 
    logTimes_disingenuous: logTimes.disingenuous,
    logTimes_bonafide: logTimes.bonafide,
    sigma_disingenuous: sigmas.disingenuous,
    sigma_bonafide: sigmas.bonafide,
    experimental: donationRates.experimental,
    control: donationRates.control,
    likert_sigma: likertSigma,
    percent_bonafide: probBonafide 
  }

}

var numSamples = 50000;

var elaborateModelPosterior = Infer({
  method: "incrementalMH", 
  samples: numSamples, burn: numSamples/2,
  verbose: true, 
  verboseLag: numSamples/10,
  model: model
})

// run a big model: takes about 1 minute
editor.put("elaborateModelPosterior", elaborateModelPosterior)

viz.marginals(elaborateModelPosterior)
~~~~

**Exercises:**


1. Explain to yourself (or a partner) why there are three `observe()` statements in the model above. What information (behavioral data) is being incorporated into the model with each of them?
2. Is there any benefit to incorporating multiple data sources into the model? Try removing (or commenting out) the observe statements corresponding to different data sources and re-running the model. (Note: If you comment out an observe statement incorporating a particular data source, you can also comment out the random variables that are only used in that observe statement.) Using the helper code below, calculate a 95% credible interval for the donation rates for these different "lesioned" models. (Since you'll be re-running the same code chunk a few times, you may want to jot down the 95% credible intervals for each run you perform.) How does the inference about the donation rates for each condition change as you incorporate different and/or more data sources? Why?
3. Can you think of a way to make this model even better?

~~~~
///fold:
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
///

// return saved results from above
var elaborateModelPosterior = editor.get("elaborateModelPosterior");

// get marginal posteriors for parameters of interest
var experimentalPosterior = marginalize(elaborateModelPosterior, "experimental")
var controlPosterior = marginalize(elaborateModelPosterior, "control")

// credibleInterval function expects a list of samples
// so generate samples from the posterior
var experimentalSample = repeat(10000, function(){ sample(experimentalPosterior)})
var controlSample = repeat(10000, function(){ sample(controlPosterior)})

display("95% credible interval for experimental condition = " + 
        credibleInterval(experimentalSample, 0.95))
display("95% credible interval for control condition = " + 
        credibleInterval(controlSample, 0.95))

// also calculate interval using kernal density smoother
// to verify consistency of results
var kdeDist = KDE({data: experimentalSample})
var kdeSamples =  repeat(10000, function(){ sample(kdeDist)})
display(credibleInterval(kdeSamples, 0.95))
~~~~

In the [next chapter](07-algorithms.html), we'll discuss different inference algorithms and when you might want to use them.

