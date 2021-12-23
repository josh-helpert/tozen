---
layout: default
title: General Purpose IR
description: 
permalink: full
---

This level adds features which are make it a full, general purpose language.

It is built on top of [Erasable IR](./erased) and is the only dependency.


## Protocols
------------------------------------------------------------------------------------------------------------

Protocols are the main interfacing and abstracting tool that is used in code commonly.
A protocols describes various types of constraints.
A type which implements a protocol is said to conform to that protocol.

```
My-Proto = Protocol
  // Relators? what if shared? enumerate?
```


## Static Containers
------------------------------------------------------------------------------------------------------------

Each of the static containers have relators where the compiler knows everything about htem.




## Variables
------------------------------------------------------------------------------------------------------------

Variables are names which are bound to values.

They represent a unit of memory.

### Names

Names follow similar conventions to records:
* start with lower alpha `a-z`
* words delimited by hypens (`my-long-var`)
  * hypens cannot start (`-my-var`) or end (`my-var-`)
* can contain numbers after start (`x-4`)
* often used in mathematics, can end with a prime suffix (`x'''`)
  * can indicate many primes by using a number after prime (`x'5` to indicate 5 primes)

### Specification



### Modifiers

Since names start a line, we need a different way to specify modifiers and constraints.

Notes:
* Maybe consider different concept of: const vs immutable vs mutable
  * const is more like comptime-only
    * there is no comptime const? or is `##x` instead of `#x`?
  * immutable is like constant-runtime?
    * which is the default
  * mutable is w/ modifier
  * look at Pony?
  * can freeze/seal at a specific point?
    * becomes immutable?
    * related to builder syntax (both runtime and comptime)?
* Differences between container modifier and value
  * can have many different aspects

```
x = 4
  :type = Int
  :mut = True

x = 4
  :type  = Int
  :const = True
```

### Interplay with Relators


## Functions
------------------------------------------------------------------------------------------------------------

Functions are the core unit of computation.

They are a container much like records, arrays, structs, and others.
In a similar way they respect specific paths.

### Input



### Output



