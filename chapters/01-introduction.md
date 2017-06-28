---
layout: chapter
title: Introducing probabilistic programs
description: "An introduction to probabilistic programs"
---

# Programming Basics

WebPPL is a small, functional programming language that looks and feels like JavaScript.
It is, in fact, a very powerful **probabilistic programming language**

As in any programing language, you can do basic arithmetical operations

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

## Arrays and Objects

Two different kinds of collections of things are arrays (sometimes called lists) and objects.
Arrays are collections of values.

~~~~
var myFavoriteNumbers = [1, 2, 3, 4, 5]
myFavoriteNumbers
~~~~

You can index in using numbers representing the position. Try: `myFavoriteNumbers[2]`

Objects have properties, and each property (sometimes called a "key") has a value.

~~~~
var myFavoriteThings = {number: 3, food: "pie"}
myFavoriteThings
~~~~

You can index in using the name of the property. Try: `myFavoriteThings["food"]` (same as `myFavoriteThings.food`)



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

~~~~

> **Exercises:**

> 1. Before running any code, calculate `g(f)`. Check your work by running the code.
> 2. Try to do the same as as `g(f)` without defining `f` explicitly.
> [Hint: Functions can be arguments of other functions. How do we represent a function?]

`repeat` is a built-in function that takes another function as an argument. it repeats it how many ever times you want

~~~~
var g = function(){ return 8 }
repeat(100, g)
~~~~

> **Exercise:** What will happen if you run `repeat(100, 8)`? Why?

# conditional statements

Another very useful thing in programming is conditional logic.

~~~~
var smellsGood = function(food){
  food[0] == "p"
};
if (smellsGood("pie")) {
  display("the world is a good place")
} else {
  display("i dont know what to believe")
}
~~~~

Javascript has a cryptic shorthand.
`(if) ? (then) : (else)`

~~~~
2 == 2 ? "yep" : "naw"
~~~~

~~~~
var f = function(x) {
  var y = x*2
  return x > 3 ? x : y
}

print(f(3))
print(f(4))
~~~~

> **Exercises:**

> 1. Before running any code, calculate `f(3)`. Check your work by running code.
> 2. Before running any code, calculate `f(4)`. Check your work by running code.

# Randomness in Programs

A probabilistic programming language is like a regular programming language with "elementary random primitives".


~~~~
flip()
~~~~

`flip` is a function like any other, so we can pass it to higher order functions like `repeat()`

~~~~
repeat(100, flip)
~~~~

WebPPL has its own basic visualization tools that are already outfitted in this website. It has a lot of functionality, whose documentation can be found [here](https://github.com/probmods/webppl-viz). The coolest thing about WebPPL viz is that it will often automatically produce exactly what you want to see, just by calling `viz()`. (It used to be called `viz.magic()`).

> **Exercises:**
>
> 1. Try calling `viz()` on the output of the above code box.
> 2. Run the code box several times. Does the output surprise you?
> 3. Try repeating the flip 1000 times, and run the code box several times. What has changed? Why?

## Distributions

So far, we have been looking at *samples* from probability distributions. The probability distribution is thus *implicit* in the sampler function. (If you repeat the sampling many times, you will start to approximate the distribution closer and closer.)

WebPPL can also represent the distribution explicitly. This is done using the capitalized versions of the sampler functions. (Note: `flip()` is a cute way of referring to a sample from the `bernoulli()` distribution.)

~~~~
bernoulli(0.5)
~~~~

Distributions have parameters. (And different distributions have different parameters.)
For example, the Bernoulli distribution has a single parameter.
In WebPPL, it is called `p` and refers to the probability of the coin landing on heads.
We can call the distributions by passing them objects with the parameter name(s) as keys and parameter value(s) as values (e.g., `{p: 0.5}`)

~~~~
Bernoulli( { p: 0.5 } )
~~~~

Try running the above box many times. Does it change? Why not?

If a distribution is explicitly constructed, it can be sampled from by calling `sample()` on that distribution.

~~~~
var parameters = { p : 0.9 }
var sweetDist = Bernoulli( parameters )
sample(sweetDist)
~~~~

The Bernoulli is the simplest of all distributions. [WebPPL has many distributions](http://docs.webppl.org/en/master/distributions.html).

What is happening in the above box?


~~~~
var geometricCoin = function(){
  ...
}

~~~~

> **Exercises:**
>
> 1. Make a function (`geometricCoin`) that flips a coin (of whatever weight you'd like). If it comes up heads, you call that same function and add 1 to the result. If it comes up tails, you return the number 1.
> 2. Pass that function to repeat, repeat it many times, and create a picture using `viz()`

In the [next chapter](02-bayesRule.html), we'll see how to use new information (e.g., observations, data) to update uncertainty in probabilistic programs.
