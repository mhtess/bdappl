---
layout: chapter
title: Analyzing Bayesian cognitive models
description: "The fully Bayesian treatment"
---


Consider a simple agent model. 

~~~~
var actions = ['italian', 'french'];

var transition = function(state, action){
  var nextStates = ['bad', 'good', 'spectacular'];
  var nextProbs = (action === 'italian') ? [0.2, 0.6, 0.2] : [0.05, 0.9, 0.05];
  return categorical(nextProbs, nextStates);
};

var utility = function(state){
  var table = { 
    bad: -10, 
    good: 6, 
    spectacular: 8 
  };
  return table[state];
};

var agent = function(state, alpha){
  return Infer({ model: function(){

    var action = uniformDraw(actions);

    var expectedUtility = function(action){
      return expectation(Infer({ method: 'enumerate' }, function(){
        return utility(transition(state, action));
      }));
    };
    factor(alpha * expectedUtility(action));
    return action;
  },  method: 'enumerate' })
};

viz(agent(null, 1))
~~~~

This agent is trying to figure out what action to take. 
It does so by calculated expected utility, which is computed via imagining how different actions would translate into utility. 
It acts according to a soft-max decision rule, which is shown in the factor statement. 

### Bayesian data analysis of a Bayesian cognitive model

The model above has a single free parameter: The soft-max parameter `alpha`. 
We could infer the credible values of that parameter, if we had data showing how the agent behaved in a given situation. 
Below, we have some example data (example decisions or actions the agent has taken), and we will build a simple BDA model to infer `alpha`. 

~~~~
var actions = ['italian', 'french'];

var transition = function(state, action){
  var nextStates = ['bad', 'good', 'spectacular'];
  var nextProbs = (action === 'italian') ? [0.2, 0.6, 0.2] : [0.05, 0.9, 0.05];
  return categorical(nextProbs, nextStates);
};

var utility = function(state){
  var table = { 
    bad: -10, 
    good: 6, 
    spectacular: 8 
  };
  return table[state];
};

var agent = function(state, alpha){
  return Infer({ method: 'enumerate' }, function(){

    var action = uniformDraw(actions);

    var expectedUtility = function(action){
      return expectation(Infer({ method: 'enumerate' }, function(){
        return utility(transition(state, action));
      }));
    };
    factor(alpha * expectedUtility(action));
    return action;
  })
};

var data = ['italian', 'french','french','french','french','french','french',
           'french','french','french','french','french','italian', 'italian',
           'french','french','french','french','french','french',
           'french','french','french','french','french','french',
           'french','french','french','french','french','french',
           'french','french','french','french','french','french',
           'french','french','french','french','french','french']

var dataAnalysisModel = function(){
  var alpha = uniform(0, 5);
  var cognitiveModel = agent('initialState', alpha);
  map(function(d){observe(cognitiveModel, d)}, data)
  return {
    alpha: alpha,
    french_prediction: Math.exp(cognitiveModel.score("french"))
  }
}

var numSamples = 5000;

var inferOpts = {
  method: "MCMC",
  samples: numSamples,
  burn: numSamples / 2,
  callbacks: [editor.MCMCProgress()],
  model: dataAnalysisModel
}

var posterior = Infer(inferOpts);

viz(posterior);
~~~~


**Exercises:**

1. Above we have returned the posterior over the parameter `alpha` as well as the posterior predictive of choosing the french restaurant. Modify the code to instead return the prior predictive of choosing the french restaurant. 
2. Try to break the model by switching around the agent's utilities and re-run the BDA model. How do you identify that the model is misspecified? 