---
layout: chapter
title: Introducing probabilistic programs
description: "An introduction to probabilistic programs"
---

# Introduction to WebPPL

WebPPL is a small, functional programming language that looks and feels like JavaScript, but is in fact, a very powerful **probabilistic programming language**

In it, you can do basic arithmetical operations

~~~~
3 + 3
~~~~

You can create variables and use them

~~~~
var x = 3
x + 3
~~~~

Like in JavaScript, numeric variables can be automatically modified into strings

~~~~
var x = 3
x + " is my favorite number"
~~~~

## Functions
The most awesome thing about programming is making functions

~~~~
var f = function(x) { return x + 3 }

var y = f(3)
y
~~~~

And the coolest thing is that functions can be arguments of other functions!

~~~~
var g = function(y) { return y(3) + 2 }
var f = function(x) { return x + 3 }

g(f)

// this can also be done without defining f explicitly

g(function(x) { return x + 3 })
~~~~

Wait what just happened?

`repeat` is a built-in function that takes another function as an argument. it repeats it how many ever times you want

~~~~
var g = function(){ return 8 }
repeat(100, g)
// why can't you do repeat(100, 8)?
~~~~

# if statements

~~~~
// (if ) ? (then ) : (else )

// 3 == 3 ? "yes" : "no"

var f = function(x) {
  var y = x*2
  return x > 3 ? x : y
}

print(f(3))
print(f(4))
~~~~

# Probability
WebPPL has distributions built into it

~~~~
// flip()

// viz.hist(repeat(1000, flip))

// sample(Bernoulli(...)) is equivalent to flip(..)
// var parameters = {"p": 0.9}
// sample( Bernoulli( parameters ) )


// sample(Discrete( {ps: [0.3, 0.4, 0.3] } ))
// sample(Categorical( {ps: [0.3, 0.4, 0.3] , vs: ["zero", "one", "two"] } ) )
viz.hist(
  repeat(
    100,
    function(){
      return categorical({ps: [1, 1, 1], vs: ["zero", "one", "two"] })
    }
  )
)
~~~~

What happened there?
First, we made a distribution `Bernoulli({p: 0.5 }) ` and then we called the function `sample` on it.

~~~~
var myParameters = {"p": 0.5}
var myDistribution = Bernoulli(myParameters)
sample(myDistribution)
~~~~

Same thing.

The nice thing about this is we can make functions with distributions!

~~~~
var coinWeight = 0.9

var coin = function(weight){
  return flip(weight) ? "Heads" : "Tails"
}

// here, we had make a new function because coin took an argument
// repeat only accepts functions with no arguments
repeat(100, function(){return coin(coinWeight)})
~~~~

Let's pass the basic coin to `repeat`. Notice that coin is a function with NO arguments. This is called a **thunk**, and is a very common type of function in functional programming.

~~~~
var coin = function(){
  return sample(Bernoulli({p:0.5})) ? "Heads" : "Tails"
}

// repeat(100, coin)
viz.hist(repeat(100, coin))
~~~~

Since functions can call other functions. Can they call themselves?

~~~~
var geometricCoin = function(){
  return flip() ? 1 + geometricCoin() : 1
}

viz.hist(repeat(1000,geometricCoin))
~~~~

> **Exercises:**

> 1. Explore what happens if you make the speaker *more* optimal.
> 2. Add another object to the scenario.
> 3. Add a new multi-word utterance.
> 4. Check the behavior of the other possible utterances.
> 5. Is there any way to get "blue" to refer to something green? Why or why not?



In the [next chapter](02-bayesRule.html), we'll see how RSA models have been developed to model more complex aspects of pragmatic reasoning and language understanding.
