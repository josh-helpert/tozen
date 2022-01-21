---
layout: default
title: Erased Sugar
description: Sugar for [Static IR](./static)
permalink: static-sugar
---

It is built on top of [Core Sugar](./core_sugar) and is the only dependency.
It emits [Static IR](./static).


## 
------------------------------------------------------------------------------------------------------------



## Types
------------------------------------------------------------------------------------------------------------

It's useful to constraint a program as precisely as possible.
Arguably the most common way is using Types.
Most languages differentiate between concepts like types, interfaces, abstract classes, traits, etc.
In Tozen, they're all just different ways to use metadata.

The metadata relator `:` allows the developer to define whatever metadata constructs they find useful outside of what's defined within Tozen syntax.
It's used for both common concepts (like types) but can be extended to more nuanced constructs like constant-ness, object-capability, concurrency, or even business logic for a certain domain.

Tozen prefers that the problem dictates the appropriate model instead of the reverse.
This means Tozen must allow the developer to extend their model to whatever they need to solve their problem.
Since often this changes for different domains and use-cases, the model must be able to be customized.

For now, we'll start with the basics and leave the more complex type-like concepts for when we fully introduce compile time.

A type must start with a alphabetic, upper-case character.
Each new word of it's name should be upper-case like `MyTypeName` (or `My-Type-Name`?).

### Type Sugar

Types get special treatement as they're so widely used and useful.

Lets start simple with an example:
```
x :(type = Integral) = 4
```

Notice that `x`'s type is literally represented by the key `type` under the metadata relator `:`.
The key `:type`, among others, has a reserved meaning in Tozen syntax to mean the logical type of the name it's describing.

Since defining types is so common, we support sugar to minimize the syntactical noise:
```
x :Integral = 4
```
This only works for identifiers which can be interpreted as a types (meaning starting with upper-case alphabetic character) and is under the metadata relator `:`.

It's still possible to add more metadata:
```
x :(Integral, min = 0, max = 10) = 4
```

or the equivalent expanded form:
```
x :(type = Integral, min = 0, max = 10) = 4
```

It is a syntax error to set both the `type` and use the shorthand together like:
```
x :(Integral, type = Decimal) = 4

// this is invalid b/c it translates to:
x :(type = Integral, type = Decimal) = 4
```


## Simple Variables (& substitution?)
------------------------------------------------------------------------------------------------------------


## Units
------------------------------------------------------------------------------------------------------------

Manually tracking units

`x_sec` vs `4_sec`?

