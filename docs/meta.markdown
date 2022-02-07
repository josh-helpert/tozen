---
layout: default
title: Metaprogramming Level
description: 
permalink: meta
---

This level adds features which supports metaprogramming.
This is the highest level which is the basis of all abstractions.

Metaprogramming is code which emits other code.
It allows us to ...TODO...

It is built on top of [General Purpose Level](./full) and is the only dependency.
It can emit any of the previous levels IR.
The code emitted will match the guarantees of that level.


## Basics
------------------------------------------------------------------------------------------------------------

Much list LISPs, the basis of metaprogramming uses the language itself.
For Tozen, these are: data, variables, and functions.
For convenience, there is also some added sugar to make the code more terse.

A developer should be familiar with metaprogramming as it functions very similarly as the general purpose language.



## Compile Time
------------------------------------------------------------------------------------------------------------

### Comptime Known Reinterpretation

Literals that are comptime known can be shared and used in multiple variables.

Since the compiler knows the value and the variable type it is able to reason about them both.
It is able to assure correctness.

A simple example:
```
// each are comptime known
#x = 1
#y = -1
#z = 1.5

// These are valid
b :Int   = #x // interpret as Int
c :Float = #x // interpret as Float which widens
d :Int   = #y // interpret as Int
e :Float = #y // interpret as Float which widens
f :Float = #z // interpret as Float which widens

// These are illegal
g :U8  = #y // Negative numbers are not in range of U8
h :Int = #z // Cannot implicitly lose information, must be explict and cast Int(#z)
```

## Codegen
------------------------------------------------------------------------------------------------------------

Notes:
* `# = ...` emit at comptime to this location
* 

## Staging
------------------------------------------------------------------------------------------------------------


## Marshalling
------------------------------------------------------------------------------------------------------------



