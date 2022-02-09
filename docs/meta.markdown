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


## Holes
------------------------------------------------------------------------------------------------------------

A common coding practice is to incrementally construct abstractions which are missing details.
Often the necessary details are provided by the contexts which use the code.
This pattern is repeated in many different language features like:
* Generics
* Parametric Polymorphism
* Python-like `pass`
* Zig-like `Undefined`
* Idris-like `holes`?
* Placeholder?

Although there are multiple manifestations, Tozen unifies them into a single concept of a `Hole`.
Conceptually `holes` are:
* comptime details missing from a program
* constructs with holes cannot be used until reified
* details must be provided before commonly used like other parts of program

### Syntax

We want the syntax to express a few properties of holes:
* its a comptime pattern
* its a missing detail which needs to be provided before used
* it must be able to be placed in nearly all locations in code (like name, value, relator, ...)
* must allow for optional names and types (much like function input)
* terse enough for common use

To match all these constraints, we have added sugar which acts like:
```
// Nameless hole value, effectively like 'Undefined'
do
  x = #?

  // Now can be used like usual
  x = 1

// Named hole value w/ constrainted type (effectively a generic)
// - When not specified it default to U8
do
  // We're missing a named value 'T' which is a Type
  MyList = List(#?T :Type = U8)

  // Supply the missing details for each type of list
  MyBoolList = MyList(Bool)
  MyIntList  = MyList(Int)
  MyU8List   = MyList()

// Parametric polymorphism
do
  sum = Fn (x :#?Type, y :#?Type) ??? or should be capture? or what?
  sum = Fn (x :#?CaptureType, y :#?CaptureType) ??? or should be capture? or what?
  sum = Fn (x :#?Capture(Type), y :#?Capture(Type)) ??? or should be capture? or what?
```

// like a comptime fn?

```
#(x :Type, y :U8)?
#(x :Type)?
#(x)?
#x? // requires a name? or #Undefined?

#?(x :Type, y :U8)
#?(x :Type)
#?(x)
#?x
```



Notes:
* which is the best name?
  * hole, hole, undefined, gap, placeholder?
* syntax:
  ```
  a???
  ???a
  ?(a)
  ?a
  #(a)?
  #?(a)
  a = Unknown // This even work b/c must be used all over not just name or value?
  Unknown(a)  // This more flexible b/c can wrap anything
  a#Unknown   // Like a relator which tags that it is Unknown? this can be anything?
  #a?         // Comptime hole?
  ```
* How to be used for diff types of polymorphism?
  * polymorphisms:
    * ad-hoc
      * function and operator overloading
      * static dispatch
      * eg `Add (U8, U8)` and `Add (String, String)`
    * parametric
      * generic function and generic programming
      * static dispatch
      * same algo without depending fully on the concerete type?
      * applies to data and functions
        * eg data `class List<T>`
        * eg function `<T> bool add (T, List<T>)`
      * eg Haskell `data List a = Nil | Cons a (List a)` or Java `class List<T>`
    * subtype/inclusion
      * virtual function
      * single and dynamic dispatch
      * multiple dispatch
      * predicate dispatch
      * according to Liskov substitution
        * note: what I often want is full subsitution to make a monomorphic call
      * usually resolved dynamically (ie dynamic dispatch)
        * can be static dispatch for complex things like Template Metaprogramming (and CRTP)
      * eg `Cat is Animal, Dog is Animal, talk (Animal)`
    * row polymorphism (duck typing)
      * structural typing losing type information
    * polytypism?
      * more general than polymorphic but need to R&D...
  * all polymorphism constrained to monomorphic constraints? must be able to monomorphized?
    * need more than this requirement as we need for comptime known abstraction?
      * b/c will be result of conditional or array and we won't know what the concrete type is?




### Undefined

`Undefined` means no value is bound to the name.
When a name is `Undefined` it is not usable until the compiler can determine a value is set.

When `Undefined` is not set a variable defaults to the `Zero` value.
Often this means the memory is zeroed or set to invalid memory to make errors clear.

Here how it is used:
```
x = Undefined

// comptime error b/c 'x' is 'Undefined'
a = x + 1

if (rand() > 0.5)
  x = 1
else // require this branch to be total?
  x = 2

// comptime error b/c 'x' may be 'Undefined'
b = x + 1

// 'x' can be set b/c may be 'Undefined'
// ??? if (x is Undefined) ???
x = 2

// Now this works b/c 'x' is known to be set
c = x + 1
```

Q:
* How do `Undefined` get propogated to: other functions, nested scopes, other?
  * Should be illegal so unable?
    * or only allowable at comptime?
* How does `Undefined` differ from something like a hole? could they be unified?
  * in that they can't be use, propogate, are types, and must be made concrete before use?

