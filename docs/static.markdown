---
layout: default
title: Static IR
description: 2nd layer of Tozen language using only statically analyzable abstractions
permalink: static
---

This level adds features which can be statically analyzed.
The compiler knows all (or nearly so) about these abstractions so it able to provide strong guarantees.
Often the abstractions are completely erased.
For a developer, this means they can freely use these tools with less worry.

It is built on top of [Core IR](./core) and is the only dependency.


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

### Static Container Protocol

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
* `:elems` is the a description of the names and types for each field
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


## Simple Rules
------------------------------------------------------------------------------------------------------------

Often developers must manually check the rules for the system they have constructed.
Many of these rules are implicit expectations, checked inconsistency, and specific to a domain.

To do better, we have a rule system which works on multiple aspects of code.
At this level, we sill focus on those which can be statically analyzed.

### Code Rules

Notes:
* variable usage patterns
  * #use, #shadow, ...
  * #linear/#use-once, ...
* local patterns
  * 
* ordering
  * invoke
  * call
  * setup
* 


## Added Knowledge
------------------------------------------------------------------------------------------------------------

* Custom Numeric
  * Whole: 0, 1, ...
  * Natural: 1, 2, ...
* Abstract types and compatiblity of codegen
  * eg Integral compatibiltiy
    * substitute w/ caller types to monomorphize?
  * Substitute/generic?
    * any type of abstract can be deal when subset is substituted?
      * an algo true for all Int is valid for any subset of ints?
        * not true, b/c can overwhelm subset of ints!
* Operators
  * Issues w/ operators?
* Range
* Inductive Types
* Rules
  * Associative
  * Commutative
  * Transitive
  * Distributive
  * Linear
  * add other usage constraints?
    * eg can use once
    * eg must use once
    * eg shadow patterns
      * eg must be a new variable each usage
* Structures
  * 
* Types w/ guarantees?
* 

## Functions
------------------------------------------------------------------------------------------------------------

Functions are special types of containers.

### Syntax

Many languages use keywords (or compiler reserved words) to inspect and define elements of a language.
Instead of that approach, functions use relators to define and describe themselves.

Although there are many relators are defined; a few describe the most common cases:
* `Fn:in` is the input of the function
  * Often is a record but will eventually be more complex
  * Requires a name for each field of the record
  * Values of a record are interpreted as its default value
* `Fn:out` is the output of a function
  * Often is a record but will eventually be more complex
  * Names are optional for each field of the record
  * Values are not allowed as the body must define which value it returns
* `Fn:body` is where the function does most of its work and executes
  * Introduces a block over its entire set of children
  * Interprets each child as a step

Consider a simple example:
```
sum = Fn()
  :in
    x :U8 = 1
    y :U8 = 2
  :out
    U8
  :body
    sum:in/x + sum:in/y
```
Notice that:
* The function must be named and unique.
  * This is generally true for all IR as must be explicit.
  * Even for lambda/anonymous functions a name must be generated.
* `Fn:out` records don't require a name.
  * When a name is used, it can be used within the function body to simplify return syntax.
* The last expression of `Fn:body` is the return value.
  * There is a `Fn:body/return` keyword within `Fn:body` which also allows for explicit returns.
* In `Fn:body` the arguments are fully qualified.
  * Explicit is better and easier for IR
  * This is even more necessary once lambda/anonymous functions are introduced.

### Invoke

Functions are invoked like many other languages.

There are some rules to consider:
* Arguments are records.
* Arguments can be named or just values.
* It is illegal to reuse the same name.
* It is illegal to use a name which isn't defined in the function.

Here are examples of invoking the previously defined function:
```
// Valid
z = sum(1, 2)
z = sum(x = 1)
z = sum(y = 2)
z = sum(x = 1, 2)
z = sum(1, y = 2)
z = sum(x = 1, y = 2)

// Illegal
z       = sum(x = False)           // 'x' type doesn't match 'sum' definition
z       = sum(x = 1, y = 2, x = 3) // 'x' isn't unique
z       = sum(z = 3)               // 'z' isn't defined in 'sum'
z :Bool = sum(1, 2)                // 'z' type doesn't match 'sum' definition
```

### Using `in` and `out`

Instead of constructing arbitrary names to represent input and output, a developer can use the `:in` and `:out` relators.

Consider a extending the sum function which also detects overflow:
```
// Definition
sum = Fn()
  :in
    x :U8 = 1
    y :U8 = 2
  :out
    z           :U8
    is-overflow :Bool
  :body
    U16 tmp_sum         = sum:in/x + sum:in/y
    sum:out/is-overflow = tmp_sum > U8:max

    if (!sum:out/is-overflow)
      sum:out/z = tmp_sum

// Usage
(z, None)   = sum(1, 2)      // Ignore error flag (no error detected)
(z, is-err) = sum(1, 2)      // Store error flag (no error detected)
(z, is-err) = sum(1, U8:max) // Store error flag (error detected)
```
Notice that `:out` are:
* Used directly to for locally scoped values as well as output.
  * This makes it easier to avoid adding arbitrary names.
* Just like `:in` they must be fully qualified at the `IR` level.
* Destructured just like any other record.

### Derived

One goal of Tozen is to move more computation into the static domain.
One way this is done is with derived computations.


## Protocol
------------------------------------------------------------------------------------------------------------

Protocols are the main means of abstraction.
They describe nearly arbitrary constraints for those types which conform to them.

Protocols have several components which work in coordination:
* Definition
  * Its requirements, fields, types, kinds, etc.
* Implementation
  * Description of how a specific type (or even protocol) conform to the protocol
  * the implemention of the protocol and how it should be applied
* Validation
  * The algorithm which validates the implementation
  * Most commonly this is covariant and contravariant restrictions
* Usage (integration better name?)
  * How the protocol interacts with other constructs like memory, arrays, functions, etc.

We can't use all of the features as they are too flexible for the static constraints of this level.
For now, we'll mainly use a protocol definition and implementation.
The rest will be fully described at the [Meta](./meta) level.

### Basics

The simplest protocols use records and relators to describe a common interface.
The interface can imply shared fields, paths, functions, among other constraints.

The developer uses builtin functions in order to define and then describe a Protocol:
* `Protocol` is a function which interprets its relators as the fields of the protocol
  * For now, the input of `Protocol` is blank and is only used for sugar
  * Each field, can add arbitrary constraints based on the type
  * Some relator paths are reserved to have special semantic meaning
* `Implements` is a function which interprets its relators as the implementation of the protocol
  * The input of `Implements` is the type (or protocol) and the protocol to be implemented
  * Some relator paths are reserved to have special semantic meaning

Consider a basic example where different animals share a common interface:
```
Speak = Protocol()
  .speak
    :type = Fn

Dog  = Struct()
Cat  = Struct()
Duck = Struct()

Implements()
  :in
    Dog
    Speak
  :impl // should this be :body?
    .speak = log("bark")

Implements()
  :in
    Cat
    Speak
  :impl
    .speak = log("meow")

Implements()
  :in
    Duck
    Speak
  :impl
    .speak = log("quack")
```

### Exposing Variables and Types

```
Dog = Struct()
  .message = "bark"

Cat = Struct()
  .message = "meow"

Duck = Struct()
  .message = "quack"

Implements()
  :in
    d     = Dog
    proto = Speak
  :impl
    .speak = log(d.message)

Implements()
  :in
    c     = Cat
    proto = Speak
  :impl
    .speak = log(c.message)

Implements()
  :in
    d     = Duck
    proto = Speak
  :impl
    .speak = log(d.message)
```

### Protocols of Protocols



Notes:
* 

### Prerequisites



### Added Constraints
