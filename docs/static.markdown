---
layout: default
title: Static IR
description: 2nd layer of Tozen language using only statically analyzable abstractions
permalink: static
---

This level adds features which are guaranteed to be erased by the compiler.
For a developer, this means they can freely use these tools without worry of overhead.
If only this level (or below) is used, then we can provide even more guarantees.

It is built on top of [Core IR](./core) and is the only dependency.


## Simple Protocol
------------------------------------------------------------------------------------------------------------

Protocols are the main means of abstraction.
They describe nearly arbitrary constraints for those types which conform to them.

We can't introduce all of the features as they are too flexible for the constraints of this level.
Instead, we'll introduce the basics and fully describe them once we reach the [Meta](./meta) level.

### Basics

The simplest protocols use records and relators to describe a common interface.

### Compatibilty

* and costs
* 

### Static Containers

All of the statically sized containers implement the same protocol.
So far, static containers will include: Record, ArrayLiteral `[]`, MapLiteral `{}`, Array, and Struct.

The compiler supplies most of the details as it's able to know nearly everything about static containers.

Static containers define the following relator paths:
* `:init` are the values used for initialization
  * initialized values are different from runtime data in several ways but beyond this level
* `:type` is the fully qualified type
  * this is set by the compiler
* `:capacity` is max size allocated
  * this is set by the compiler but the allowed to override
* `:first` is the first syntactical element when was first constructed
  * this is set by the compiler
* `:last` is the last syntactical element when was first constructed
  * this is set by the compiler
* `.len` is the current number of elements
  * In general, `.len` is always <= `:capacity`
  * this is set by the compiler
* `/{UInt}` is to access a specific element
  * Each child of a static container can be accessed by using its index (0-based)
  * The `UInt` value will throw a comptime error if it can determine it will exceed the bounds set in `:capacity`

Consider how these apply to a simple record:
```
r = (a = 1, '2', c = True)

r:init     // (a = 1, '2', c = True)
r:type     // Record(Int, VerbatimString, Bool)
r:capacity // 3
r:first    // 1
r:last     // True
r.len      // 3
r/0        // 1
r/1        // '2'
r/2        // True
```


## Simple Variables
------------------------------------------------------------------------------------------------------------

At this level, variables are not much different from mapping a name to a value.
In order to allow the compiler to better reason about your code, variables are constant.
Later levels will allow for more flexibility like mutability.

### Scope


### Shadow






## Simple User Types
------------------------------------------------------------------------------------------------------------

Users are able to create their own types using builtin functions.
These can become complex as you add generics, polymorphism, refinement types, dependent types, predicates, capabilities-secure, traits, etc.

These can become very rich and complex but they are more closely related to metaprogramming.
Technically all types are just functions with data at the metaprogramming level.
That is too complex for this level.
For now, we constrain the user types to a simple subset until we fully introduce metaprogramming level.

These abstractions are fully erased by the compiler.
The compiler must know everything about them.
This means they must be:
* immutable
* statically sized
* type information is fully specified

A benefit to the developer is that they're completely free to use.

### Array-like

The `Array` type is very similar to the Array literals `[]`.
`Array` type is more flexible and doesn't require us to use inference.

To construct an explicit `Array` a developer just constructs it directly as a function.
Then use relators in order to describe the fields and structure that is relevant to an `Array`.

Array implements the static container protocol from above.
It also adds its own:
* `:elem-type` is the type of each element
  * this is set by the compiler but the allowed to override

Here is an example which uses all the fields that can be set:
```
arr = Array()
  :init      = (1, 2, 3)
  :elem-type = Int
  .capacity  = 5
```

Using the Array literal syntax this is equivalent to:
```
// When not specified, will default to Zero of Int
arr = [ 1, 2, 3, 0, 0 ]
```

If we query all of the information from the array we find:
```
arr:init      // (1, 2, 3)
arr:elem-type // Int
arr:type      // Array(Int)
arr:capacity  // 5
arr:first     // 1
arr:last      // 0
arr.len       // 3
```

### Struct-like

Building on Record syntax; we use the builtin type `Struct` to declare a new `Struct`-like type.

Struct implements the static container protocol from above.
It also adds its own:
* `:elems` is the structure of the names and types for each field
  * this is simply a record without values
  * this is set by the compiler but the allowed to override

Here is an example which uses all the fields that can be set:
```
// TODO:
// - Isn't this invalid b/c 'a = None'? should be diff value or allow in general? is empty record? meaningful?
// or include some type of placeholder syntax? or any-value syntax? or default/empty value?

s = Struct()
  :init     = (a = 1, b = '2', c = True)
  :elems    =
    a
      :type = Int
    b
      :type = VerbatimString
    c
      :type = Bool
```

Using the Map literal syntax this is equivalent to:
```
s = { a = 1, b = '2', c = True }
```

If we query all of the information from the struct we find:
```
s:init      // (a = 1, b = '2', c = True)
s:type      // Struct(a :type = Int, b :type = VerbatimString c :type = Bool)
s:capacity  // 3
s:elems     // (a :type = Int, b :type = VerbatimString c :type = Bool)
s:first     // 1
s:last      // True
s.len       // 3
```


## Simple Functions
------------------------------------------------------------------------------------------------------------

Functions are containers much in the same way that `Array` and `Struct` are.

In order to the compiler to reason about user functions, they must conform to a few constraints.
They must be:
* total
* associate?
* commutative?
* deterministic?

### Syntax

```
sum = Fn()
  :in
    x :Numeric
    y :Numeric
  :out
    z :Numeric

sum(1, 2)
sum(x = 1, 2)
sum(1, y = 2)
sum(x = 1, y = 2)
```

### Constraints

* forall
  ```
  // increment
  example : forall (x : Natural) -> Natural
    x + 1

  \(x : Natural) -> x + 1

  // builtin length
  List/length : forall (a : Type) -> List a -> Natural
  forall (a : Type) -> List a -> Natural
  forall (b : Type) -> List b -> Natural
  forall (elementType : Type) -> List elementType -> Natural

  // builtin map
  List/map
    : forall (a : Type) -> forall (b : Type) -> (a -> b) -> List a -> List b


  ```
* 

### Builtin

* Based on specific types/proto
  * filter, map
  * transform
  * concat
* Specific types
  * Record
  * Array
  * Struct


## Added Knowledge
------------------------------------------------------------------------------------------------------------

* Custom Numeric
  * Whole: 0, 1, ...
  * Natural: 1, 2, ...
* Range
* Inductive Types
* Rules
  * Associative
  * Commutative
  * Transitive
* Structures
  * 
* 
