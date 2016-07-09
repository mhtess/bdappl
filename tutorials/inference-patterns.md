# Patterns of inference

Name of the game in these examples is to understand what effect manipulating `b` will have on `a`.

That is, if we change `b`, how will `a` change?

## Statistical dependence

~~~~
var model = function(observation){
  var c = flip(0.5)
  var b = flip(0.5)
  var a = b ? flip(0.9) : flip(0.2)
  condition(b == observation)
  return {"a": a}
}
var b_is_true = Infer({method: "rejection", samples:1000}, 
     function(){ model(true) })

print("We observe b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:1000}, 
     function(){ model(false) })

print("We observe b is not true")
viz.auto(b_is_false)
~~~~

~~~~
var model = function(observation){
  var c = flip(0.5)
  var b = c ? flip(0.1) : flip(0.9)
  var a = c ? flip(0.1) : flip(0.9)
  condition(b == observation)
  return {"a": a}
}
var b_is_true = Infer({method: "rejection", samples:1000}, 
     function(){ model(true) })

print("We observe b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:1000}, 
     function(){ model(false) })

print("We observe b is not true")
viz.auto(b_is_false)
~~~~

## Screening off

~~~~
var model = function(observation){
  var c = flip(0.5)
  var b = c ? flip(0.1) : flip(0.9)
  var a = c ? flip(0.1) : flip(0.9)
  condition(b == observation)
  return {"a": a}
}

var b_is_true = Infer({method: "rejection", samples:10000}, 
     function(){ model(true) })

print("We observe b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:10000}, 
     function(){ model(false) })

print("We observe b is not true")
viz.auto(b_is_false)
~~~~

Now, what happens if we know `c` to be `true` ?

~~~~
var model = function(observation){
  var c = flip(0.5)
  var b = c ? flip(0.1) : flip(0.9)
  var a = c ? flip(0.1) : flip(0.9)
  condition(c & (b == observation))
  return {"a": a}
}

var b_is_true = Infer({method: "rejection", samples:10000}, 
     function(){ model(true) })

print("We observe c and that b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:10000}, 
     function(){ model(false) })

print("We observe c and that b is not true")
viz.auto(b_is_false)
~~~~

## Explaining away

~~~~
var model = function(observation){
  var a = flip(0.5)
  var b = flip(0.5)
  var c = (a || b) ? flip(0.9) : flip(0.2)
  condition(b == observation)
  return {"a": a}
}

var b_is_true = Infer({method: "rejection", samples:10000}, 
     function(){ model(true) })

print("We observe b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:10000}, 
     function(){ model(false) })

print("We observe b is not true")
viz.auto(b_is_false)
~~~~

~~~~
var model = function(observation){
  var a = flip(0.5)
  var b = flip(0.5)
  var c = (a || b) ? flip(0.9) : flip(0.2)
  condition(c & (b == observation))
  return {"a": a}
}

var b_is_true = Infer({method: "rejection", samples:1000}, 
     function(){ model(true) })

print("We observe c and that b is true")
viz.auto(b_is_true)

var b_is_false = Infer({method: "rejection", samples:1000}, 
     function(){ model(false) })

print("We observe c and that b is not true")
viz.auto(b_is_false)
~~~~