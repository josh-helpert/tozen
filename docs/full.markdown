---
layout: default
title: General Purpose Level
description: 
permalink: full
---

This level adds features which are needed for a full, turing-complete language.

It is built on top of [Data Level](./data) and is the only dependency.

It's header specification is `(lang = 'Tozen', level = 'execute')`. ???


## Overview
------------------------------------------------------------------------------------------------------------

The core of Tozen are a few concepts which work together:
* data (variables, relators, data-structures)
* functions
* metaprogramming
* hard-coded information (syntax sugar, primitives, known data sets, proofs)

These work in coordination as the basis of coding.
Often by interpreteting data according to its metaprogramming type.

This level introduces more data and functions with its own hard-codec information.
However each concept is interlinked and is not able to fully disentangle.
For example, a `Fn` is actually a metaprogramming abstraction but for practical purposes it can be treated as a general purpose type.

### By default, no runtime

A developer can't implicitly use abstractions which have a runtime cost.
This means abstractions which are common in other languages are not the default choice.
Structures like: closures, generators, resizing arrays, subtype polymorphism, abstract lists, and many more.

This requirement is in place for a few reasons:
* Tozen is meant for many environments (from drivers to websites).
  It cannot enforce high-level flexibility at the cost of low-level control.
* Implicit costs paired with lack of control can result in less predictable code
* Simpler constructs are easier to optimize
* Developers can do much of their work using constrained abstractions.
  I think many will be surprised how few they need when it is not the default.

This doesn't mean developers can't use constructs like these.
Rather a developer must be explicit in their intent by opting-into using them.
It also makes developers aware that certain abstractions have costs which need to be considered.

Once a developer accepts the cost; there are many ways to use them.
The most common way is simply defining how to implement an abstraction as often there are multiple implementations.
Additionally the standard library includes constructs which allow runtime costs.

### Compile Time Basics

Metaprogramming is the main means of abstraction as it is often zero cost.
Compile time (aka comptime) is a part of metaprogramming.
Understanding the basics of comptime is essential to understanding the syntax of the Tozen language.

The necessary basics are:
* comptime prefix `#`
  * To require that a name belongs to comptime it is prefixed with `#`
    * This can be attached to many constructs like: names, functions, operators
  * modifier?
* Upper-case identifiers
  * Often upper-case identifiers indicate comptime constructs for a `Type` or `Protocol`
* comptime known
  * These are variables (and other constructs) which can be understood by the compiler using static analysis.
  * This is very similar and inspired by Zigs comptime known concept.
    Although we do metaprogramming differently this aspect is the same.
  * Often these include constructs like:
    * literals: `4e2.5`, `True`, `None`
    * comptime variables: `#x`, `#do`, `#sum`
    * upper-case identifiers: `Fn`, `Type`, `Protocol`
    * builtins: `do`, `if`, `loop`
    * constants: `Pi`, `-Inf`, `Inf`
    * hard-codec concepts: `Natural`, `Whole`, `Imaginary`

This is enough to understand the rest of this guide.
A thorough overview will be in the next layer for metaprogramming.

### Execute using `do` and `#do`

Everything in Tozen is data.
Data is inert by default.

To treat data as a block, builtin functions are provided.
The primary functions are `do` (for runtime execution) and `#do` (for comptime execution).
These functions interpret their children as steps to execute as a block.

Curly braces `{}` are not used for scopes.
Instead, the `do`-family of functions are used.

`do` block are executable blocks and can be treated like a normal block in many other languages:
* are not looping
* reduces need for nested conditionals
* can be treated as an expression
* can be broken out of
  ```
  do
    sum = 1 + 2

    if sum == 3
      break -1
  
    sum    
  ```

`do` blocks are not looping can be broken out of like:
```
```

For example, this is how functions treat their `Fn:body` as a executable block.

### Relators instead of hard-coded syntax

Many languages use hard-coded syntax to define every feature of a language.
Tozen does as well but tries to minimize the need to memorize syntax.

Instead of custom syntax, each abstraction defines relators which are meaningful for that abstraction.
This has benefits beyond the need to continually define more syntax like:
* simplifies extension by new abstractions
* minimizes the need to introduce new syntax as extension can often be handled by new relators
* the value of each relator are data (just like all the syntax) which developers already know how to read
* developers intuitively understand data which lowers the ability to reason about new extensions
* relators map to the AST representation nearly directly.
  This means metaprogramming and general purpose programming use the same representation.

For example, a function `Fn` defines the relators (and many more):
* `Fn:in` is interpeted as input arguments
* `Fn:out` is interpeted as output values
* `Fn:body` is interpreted as executed function block

### Pattern

To make code understandable for humans, we must balance terseness with explicitness.
Too much terseness and the developer is memorizing special cases, sigils, and code becomes unreadable.
Too much explicitness and the developer is lost in the details unable to see the general flow of the code.

To create this balance, patterns are introduced.
The patterns must be understandable by the compiler.

Notice that patterns only apply to the ergonomic syntax portion of the language.
The IR portion is explicit, unambigous, and consistent.



## Variables
------------------------------------------------------------------------------------------------------------

Variables are a name bound to value.

### Syntax

Variables are an interpretation of a record which:
* interprets record names as variable identifiers
* interprets record values according to the natural interpretation:
  * if value is a name, get the value
    ```
    x = 1 // Define 'x' as primitive
    y = x // Get value of 'x'
    ```
  * if value is executable, execute it, then get the return value
    ```
    add = (x, y) -> x + y // Define lambda
    sum = add(1, 2)       // Execute 'add' and get return value
    ```
  * otherwise treat as value
    ```
    x = 1 // Define 'x' as primitive
    ...TODO: more...
    ```

### Scope

Variables are defined within a scope.

Scopes nest in a similar way to other languages using lexical scoping rules.

A simple example:
```
x = 1

// x is visible w/n nested block
do
  y = x + 1

  // x, y are visible w/n nested block
  do
    z = x + y

log x // 1
log y // error b/c not in scope
log z // error b/c not in scope
```

### Levels of mutable

// TODO: Mutable semantics? depend on context?

Mutable variables are flexible yet risky.
The compiler knows the least about them and thus can provide the fewest guarantees.

We attempt to balance the need for safety with clarity.
To do so, we divide mutability into different levels of risk:
* mutable until used: mutable until first used by any other block
  * useful for avoiding need to track down variables and usage b/c can only be changed by local scope
  * otherwise immutable
* current scope mutable: mutable only in current scope
  * useful for avoiding tracking down nested mutations of variable
  * otherwise immutable
* current and nested scopes mutable: mutable only in current and nested scopes
  * useful for allowing locally mutable state (not shared)
    * means we don't need to hunt down in other scopes
  * is the default when variables are in global or module scope
  * otherwise immutable
* generally mutable

A simple example which uses each type of mutability:
```
// Declare levels of mutable variables
x #mut(#current) = 1 // declare 'x' as allowing current scope mutation
y                = 1 // declare 'x' as allowing current and nested scope mutation
z #mut           = 1 // declare 'x' as allowing general mutation

// Same scope as declarations
x = 2 // allowed
y = 2 // allowed
z = 2 // allowed

// Create nested scope
do
  x = 3 // illegal
  y = 3 // allowed
  z = 3 // allowed

// Same scope after nested scope
x = 4 // allowed
y = 4 // allowed
z = 4 // allowed

// Mutate functions within a function
f = Fn ()
  x = 5 // illegal
  y = 5 // illegal
  z = 5 // allowed

// Create function which uses current values
// - all allowed b/c val is copied not by reference
// - mutation to variable occurs in current scope
g = Fn (val)
  val + 1

x = g(x) // allowed
y = g(y) // allowed
z = g(z) // allowed
```

### Immutable and Constant

Global, module, and shared variables default to immutable.
Immutable is more constrained than mutable.
Immutable variables simply mean they are unchanging at runtime.

A simple example:
```
// Declare at module at file or module scope
x = 1

// comptime error b/c 'x' is immutable
x = 2

// Attempt mutate within a nested scope
do
  x = 3 // comptime error b/c 'x' is immutable

// Attempt mutate within a function scope
f = Fn ()
  x = 4 // comptime error b/c 'x' is immutable
```

Constant is more constrained than immutable.
Constants are not only immutable but also must be comptime known.
Often this means they must be a literal or refer to a comptime value.
The `#` prefix is used b/c `#` is used to indicate comptime.
A simple example:
```
// Allowed
a  = 1 // normal immutable variable which is also comptime known (b/c of literal)
#b = a // allowed b/c `a` is comptime known

// Illegal
c  = rand() // normal immutable variable (notice not comptime known b/c rand() isn't knowable)
#d = c      // illegal b/c 'c' is not comptime known
#e = c:type // allowed b/c 'c:type' is comptime known
```

### Modifier

There are numerous modifiers.
The difficulty of visually parsing modifiers grow in proportion to the implementation complexities and number of domains grow.
This is one reason names nearly always start a line b/c it can become difficult to visually track a mix of modifiers, names, and values.

Modifiers are a type of metaprogramming and thus use the `#` prefix.
They also have two forms inline and multiline.

Consider a somewhat complex variable declaration which is integral, mutable, thread-local, and atomic:
```
// multiline form
x = 1
  :type         = Int
  #mutable      = True
  #thread-local = True
  #atomic       = True

// inline form
x :Int#(mut, thread-local, atomic) = 1
```

Notice that we can tag specific elements using a `#` suffix.
Tags are only only used inline.
It is often useful to make it explicit which value is being described.
As the complexities of signatures increase this becomes more important.

As the number of modifiers grow these too become cumbersome to write out.
There is a metaprogramming type which understands how to interpret modifiers.
Let us simplify:
```
// My own modifier which collects multiple modifiers
#my-mod = Modifier
  :type         = Int
  #mutable      = True
  #thread-local = True
  #atomic       = True

// Expands to the same set of modifiers
x #my-mod = 1
y #my-mod = 2
z #my-mod = 3
```
This will be fully explained in the next layer for metaprogramming.

### Global and Module Order Resolution

Assuring every global and module variable is declared and initialized in order is drugery without merit.
This complexity grows as more modules are shared.

To avoid this annoyance, global and module variables are automatically resolved using a dependency graph.

A simple example which works for global and modules:
```
msg-len = msg.len
msg     = "you are \(age) years old"
age     = 45
```

### Casting and Coercion





Q:
* mutable until value is `get`?
* coersion?
  * ever allowed? even in comparison? widening?
  * only when no information is lost? look at Zig?
  * what about comparison (eg U16 > U8)? peer type resolution?
* usage patterns
* modifier: resuse and declare to dry up?
  * wait for metaprogramming?
* derived vs reactive
* family types
  * or types of same family which we know are the same but subsituted?
* infinite types
  * diff types of resolutions
  * like type plus exension?
* How to model complex and nested structures
  * need different aspects to be modified
  * should be moved to meta? or later section?
* resolution at module level to avoid annoyances
* use modifier types which can collect multiple constraints together?
  * this way we can just declare all the modifiers as data and then apply them to specific names/variables?
    * or a feature of meta level?
* Narrowing allowed?
  * when does this work and not?
* Immutable and Constant
  Q:
  * downside of further narrowing of constraints is have to look in multiple places?
  * should `#mut` be more like a `Ref` type?
    * and then mutation is through that reference?
    * or this is more of a container?
      * a way to do this w/o a container concept? comptime pointer/ref?
* Seal and Constrain
  Q:
  * Should this be allowed or does it complicate too much?


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


## Shadowing
------------------------------------------------------------------------------------------------------------

By default, shadowing is not allowed.
If a dev chooses to shadow they must do so with additional syntax indicating their intent.

Shadowing is restricted in multiple ways to make it more consistent.

There are 3 levels to declare shadowing: global, block, and name specific.
The tools available for shadowing vary based on the level.

### Block Shadowing

Block shadowing affects the entire block so it's easier to track which names are shadowed.
There are a few compiler directives:
* `#use` isolates supplied names within a block from the scope it's defined in
  * Creates a new, isolated scope
  * Doesn't allow shadowing of the included names from the outer scope, but all else can be
  * Requires names to be provided
  * Allows developer to reason locally while having a single place to see the listing of names
  * Adapted from Brian Will's syntax (which itself was adapted from Jonathan Blow's Jai language)
  * Final values can be returned as result
* `#isolate` is fully isolated block
  * Instead of requiring names, it doesn't allow them
  * The block is fully isolated from the scope it's defined in
  * Final values can be returned as result

A simple `#use` example:
```
x :Int = 1
y :Int = 2
z :Int = 3

// z is available for shadowing because not in list of #use input
#use x, y
  z :Float = 4.0

// x, y is available for shadowing because not in list of #use input
#use z
  x :Float = 5.0
  y :Float = 6.0

// comptime error b/c #use must supply names
#use
  x :Float = 5.0
  y :Float = 6.0
  x :Float = 7.0
```

A simple `#isolate` example:
```
x :Int = 1
y :Int = 2
z :Int = 3

// all names are available for shadowing because isolated scope
#isolate
  x :Float = 5.0
  y :Float = 6.0
  x :Float = 7.0

// comptime error b/c #isolate doesn't allow names
#isolate x, y
  x :Float = 5.0
  y :Float = 6.0
  x :Float = 7.0
```

### Name Shadowing

Name specific shadowing alters the behavior of a name making it's shadowing behavior more consistent.
There are a few compiler directives:
* `#unique` requires that no name within the same module can shadow it
  * It is treated as a module unique value and is enforced by the compiler
  * This makes it very consistent as names become uniquely defined within their modules
* `#shadow` requires that each assignment is a new declaration.
  * Each assigment shadows each previous one.
  * This captures a few common patterns which otherwise require manual tracking of shadowing or mutation.

A simple `#unique` example:
```
x :Int #unique = 1

#use
  x :Int = 2 // Will throw a compile time error because x can't be shadowed
```

A simple `#shadow` example:
```
ret :Int #shadow = 0

// ret is always shadowed on assignment
ret = f()
if (ret < 0) return ret;

// ret is always shadowed on assignment
ret = g()
if (ret < 0) return ret;
```

### Alternatives

Even better than shadowing is using other tools to perform the same job.
If those tools are terse it makes them a viable alternative.

Here are some of the tools Elder provides which can replace some common use-cases shadowing is often used for:
* `do` executes a block and then returns the result.
  ```
  x = do
    a  = 4
    a += f(a)
    a *= g(a)
    a  = h(a)
    a
  ```
  Instead of shadowing can run a block with a mutable variable and just assign it to a immutable value.
* Piping (`|>` syntax) makes it easier to process a sequence of steps.
  ```
  x = 0 |>
    + 1
    * 2
    % 3
  ```
  Instead of using shadowing to explicitly pass a name can pipe each operation
* Versioning comptime directive requires one to version depending on a specific change
  ```
  x #version(#shadow) = 1
  x'                  = 2
  x''                 = 3
  ```
  Here each version of `x` share a similar name but we use priming `'` to indicate a new version.
  * A developer can do this manually but asking the compiler to enfoce it for you makes it a guarantee.
  * Several syntaxes are supported to indicate a new version like `x'`, `x2`, `x-2`, `x_2`
  * Note that this is a general compiler tool to support multiple aspects of code like: `#version(#shadow), #version(#mutate), #version(#shadow or #mutate)` and really deserves it's own discussion.


## Array
------------------------------------------------------------------------------------------------------------

Array literals describe values using brackets `[]`.
Fixed size arrays are an extension of array literals.
They also use brackets `[]` but as sugar to describe types. 

The array size and any names are constant.
The array elements follow the variable default, but can be made mutable or const.

A simple example:
```
// Multiline version
arr = [0, 1, 2]
  :type      = Array(U8)
  :elem-type = U8
  .len       = 3
```

This has multiple forms of sugar as well which will infer the details for you:
```
arr :[3]U8 = [0, 1, 2] // explicit sugar type description
arr :[]U8  = [0, 1, 2] // infer len
arr :[3]   = [0, 1, 2] // infer type (will infer as Range(0,2)? or (0, 1, 2)? or U8? mem vs logical type?)
arr        = [0, 1, 2] // infer len and type
```
Notice that:
* The `Type` sugar uses the array literal syntax `[]`
* The `Array.len` is within the brackets `[]` while the `Array.elem-type` is after
* All specifiers are optional for sugar and will infer the most constrained version it can
  * When inference isn't possible a comptime error will be thrown

### Multidimensional

Multidimensional arrays work in a similar way:
```
// Multiline version
arr = [[0, 1], [2, 3], [4, 5]]
  :type      = Array(Array(U8))
  :elem-type = U8???
  .len       = (3, 2)
```
Notice that:
* `Array.len` is a record which describes the lengths from outer to inner
* `Array.elem-type` refers to ??? (U8 or Array(U8) or what?) ???

This has multiple forms of sugar as well which will infer the details for you:
```
arr :[3][2]U8 = [[0, 1], [2, 3], [4, 5]] // explicit sugar type description
arr :[][]U8   = [[0, 1], [2, 3], [4, 5]] // infer len
arr :[3][2]   = [[0, 1], [2, 3], [4, 5]] // infer type (will infer as Range(0,5)?)
arr           = [[0, 1], [2, 3], [4, 5]] // infer len and type
```

### Modifiers

Although array structures are fixed sizes and structure, their contents can be mutable or const.

To make the array elements mutable, tag them with `#mut`:
```
// this looks like mutating array not content?
arr :[]U8#mut = [0, 1, 2]
```

To make the array elements constant (and comptime known), tag them with `#const`:
```
arr :[]U8#const = [0, 1, 2]
```

### Access

Just like records, the elements of an array are it's children.
Children are accessed using the `/` relator by name or index.

A simple example of access and mutating an array:
```
arr :[]U8 = [x = 0, 1, 'my name' = 2]

// Get each value by index and name
arr/0         // 0
arr/1         // 1
arr/2         // 2
arr/x         // 0
arr/'my name' // 2

// Mutate
arr/0         = 3
arr/'my name' = 4

// After mutate
arr/0         // 3
arr/2         // 4
arr/'my name' // 4
```

### Constant Names

Names are allowed in arrays.
They must always point at the same position in the array even when the array is mutable.

A simple example:
``
arr #mut = [x = 3, 4, z = 5]

arr/0 = 6
arr/2 = 7

log(arr) // [x = 6, 4, z = 7]
``

### Psuedo Dynamic

Notes:
* pair w/ holes
* can delay use of fields until later w/ holes
* only fields usable when determined at comptime can be used
  * all holes must be filled till only those fields can be used (but rest can?)
* 

### Invalid

There are various illegal forms, which the compiler can statically determine, and will throw a comptime error.

This is to prevent obvious errors like:
* `Array.len` is less than number of values
  ```
  // comptime error b/c: 2 < 3
  arr :[2] = [0, 1, 2]
  ```
* 


Notes:
* Arbitrary extension until use?
  * copy when needed? or other resolution?
* holes
* copy? reference?
  * container vs conent?
* 


## Struct
------------------------------------------------------------------------------------------------------------

Struct literals describe values using curly braces `{}`.
Fixed size structs are an extension of struct literals.
They also use brackets `{}` but as sugar to describe types. 

The struct size and names are constant.
The struct elements follow the variable default, but can be made mutable or const.

A simple example:
```
// Multiline version
struct = {a = 'x', b = 1, c = True}
  :type      = Struct(VerbatimString, Integral-Literal, Bool)
  :elem-type = (VerbatimString, Integral-Literal, Bool)
```

This has multiple forms of sugar as well which will infer the details for you:
```
struct :{VerbatimString, Integral-Literal, Bool} = {a = 'x', b = 1, c = True} // explicit sugar
struct                                           = {a = 'x', b = 1, c = True} // infer type
```
Notice that:
* The `Type` sugar uses the array literal syntax `{}`
* `Struct.len` is inferred and not settable by user

### Nested

Nested structs work in a similar way:
```
struct = {t = {a = 'x'}, u = {b = 1, c = True}, v = {d = 1.1, e = False, f = "z"}}
  :type =
    Struct(VerbatimString)
    Struct(Integral-Literal, Bool)
    Struct(Decimal-Literal, Bool, String-Literal)
```
Notice that:
* The `:type` is a record which describes each of the structure of each struct

This has multiple forms of sugar as well which will infer the details for you:
```
// All inline and inferred
struct = {t = {a = 'x'}, u = {b = 1, c = True}, v = {d = 1.1, e = False, f = "z"}}

// Somewhat multiline and inferred
struct = {
  t = {a = 'x'}
  u = {b = 1, c = True}
  v = {d = 1.1, e = False, f = "z"}

// Fully multiline and inferred
struct = {
  t = {
    a = 'x'
  u = {
    b = 1
    c = True
  v = {
    d = 1.1
    e = False
    f = "z"
```

### Modifiers

Although Structs are fixed sizes and structure, their contents can be mutable or const.

To make the elements mutable tag them just with variables:
```
// Infer type and mutable
struct #mut = {a = 'x', b = 1, c = True}

// Explicit type and mutable
// - Notice each element can be made modified
struct :{VerbatimString#mut, Integral-Literal#mut, Bool#mut} = {a = 'x', b = 1, c = True}
```

To make them constant (and comptime known) then add a `#` to the element type to indicate comptime known:
```
// Infer type and const
struct #const = {a = 'x', b = 1, c = True}

// Explicit type and const
// - Notice each element can be made modified
struct :{VerbatimString#const, Integral-Literal#const, Bool#const} = {a = 'x', b = 1, c = True}
```

### Access

Just like records, the elements of a struct are it's children.
Children are accessed using the `/` relator by name or index.

A simple example of access and mutating an struct:
```
struct :{VerbatimString, Integral-Literal, Bool} = {a = 'x', b = 1, 'my name' = True}

// Get each value by index and name
struct/0         // 'x'
struct/1         // 1
struct/2         // True
struct/a         // 'x'
struct/b         // 1
struct/'my name' // True

// Mutate
struct/0         = -1
struct/'my name' = False

// After mutate
struct/0         // -1
struct/2         // False
struct/'my name' // False
```

### Invalid




## Enum
------------------------------------------------------------------------------------------------------------

Enumerations (enums) are a way to express one of many possible variants.

This abstraction unifies multiple concepts:
* C-style enums
* C-style unions
* Java-style enums?
* Algebraic Data Types (ADT) often used if functional languages

### Syntax

To define an new enum uses the `Enum` type.
It interprets the data syntax (often records) as the variants of the enum like:
```
Color = Enum
  Red   = _
  Green = _
  Blue  = _
```
Notice that:
* Each child of Enum are its possible variants
* Each child starts with a upper-case name
* The ackward syntax is a consequence of each entry is a record.
  Record values are essential while names are optional.
* Enum names without values is quite rare.

If we want to interpret the record entries as only names you must use a metaprogramming directive.
The directive rewrites value literals as names with `None` as the value.
This is equivalent to the previous example:
```
Color = Enum
  #Names
    Red
    Green
    Blue
```

### Simple

Most often, enums will use values.

Values can be whatever a developer needs.
As simple as `Int` or as complex as a function.

Consider a simple example modeling primary colors:
```
StopLight = Enum
  Green  = 0
  Yellow = 1
  Red    = 2

StopLight = Enum
  Green, Yellow, Red == 0 .. 2
```

### Algebraic Data Type - like

Enums can be much more like algebraic data types (ADT).

Consider an example takes from OCaml website to model simple arithmetic:
```
type expr =
    | Plus   of expr * expr  (* a + b *)
    | Minus  of expr * expr  (* a - b *)
    | Times  of expr * expr  (* a * b *)
    | Divide of expr * expr  (* a / b *)
    | Var    of string       (* "x", "y", etc. *);;

(* model 'n * (x + y)' *)
Times (Var "n", Plus (Var "x", Var "y"));;
```

We can recreate with Tozen:
```
Expr = Enum
  Plus   = (Expr, Expr) // a + b
  Minus  = (Expr, Expr) // a - b
  Times  = (Expr, Expr) // a * b
  Divide = (Expr, Expr) // a / b
  Var    = String       // "x", "y", etc.

// model 'n * (x + y)'
Times(Var("n", Plus(Var("x"), Var("y")))) // all ()
Times Var("n", Plus(Var("x"), Var("y")))  // drop outermost ()?
Times Var "n", Plus Var "x"; Var "y"      // drop all ()?
```

Notice that:
* Each arm can use the parent `Enum` generalize over itself
* Each arm can accept multiple inputs
* Each arm with signatures (not literals) are a Type
* When unambigous, can use the name of each arm instead of the fully qualified name
  * eg use `Plus` when unambigous, otherwise fully pathed version is `Expr/Plus`

### Enum of Enum

Enums can be arbitrarily nested.

This allows one to create complex structures with flexible structure.

Consider a example which uses nesting and enums which depend on each other:
```
Cup = Enum
  Paper     = U8
  Wood     = Bool
  Mug = Enum
    Tea = (Inner)
    Coffee = (Outer)
    Z = (A, Y)?

enu-0 = B(false)
enu-1 = Y(
```

Notice that:
* nested enums can depend on outer, but not vice-versa
* 

### Access


### Complex

Notes:
* Java-like?
* Nested ADT
* GADT?


### Function Value?



### Reduction








Notes:
* layout
* size
* total?
* how do enum constructors differ from comptime function constructors?
* ex match?
  ```
  Shape = Enum
    Circle   = (radius :Whole)
    Triangle = (side-a :Whole, side-b :Whole, side-c :Whole)
    Square   = (side-a :Whole)

  Shape/Circle(1)
  Shape/Triangle(1, 2, 3)
  Shape/Square(1)

  ??? look at Haskell and others too ???
  - should we also match nested values?
  - destructure based on specific values or something not possible w/ GC?
  match(shape)
    Shape/Circle   = (radius)
    Shape/Triangle = (side-a, side-b, side-c)
    Shape/Square   = (side-a)
  ```




## Reference
------------------------------------------------------------------------------------------------------------

For many types of programs, full memory control isn't needed.
In these cases, a simpler abstraction can be used.
Instead of pointers one can use references (`Ref`).

References function by:
* Referring to a specific value, type, and its constraints
* Allows update to 
* 

Notes:
* Looks at another variable?
  * or is owned by the `Ref` and w/n it?
    * the `Ref` is only which is visible?
  * This different than looking at an existing variable?
* Owning is different than traversal
  * This is more like wrapping a mutable?
    * Look at ML, OCaml, and Zig to see diff types?
  * differentiate types of operations like:
    * factors:
      * singular value (eg U8) or multiple values (eg arr)
      * existing var or fully owned var
      * others?
* 


## Result
------------------------------------------------------------------------------------------------------------

Programs sometimes fail is simple, knowable ways.
Instead of a full Exception system, a simple value representation can be used.
Simple problems should have simple solutions.

To model these cases, we add a new type:
```
Result = #do (T :Type, E :Type) ->
  Enum
    Ok  = T
    Err = E
```
Notice that:
* The definition uses `#do` in order to construct a complex expression at comptime
* The `Result` is an `Enum` of 2 arms of `Ok` and `Err` to represent each case
* The `Result` type replaces other value types like `Option`, `Maybe`, `Either`.
  For more complex exception handling one must use the full exception system.

### Usage

```
// TODO:
// - Should we have unique indexes to determine categories of error?
//   - this close to Zig universal index of errors?

is-user-logged-in = Fn (username :String) -> Result(Bool, String)
  db = connect-to-db()
  if (not db) return Err("Cannot connect to db")

  user = db.find-user(username)
  if (not user) return Err("Cannot find user")

  return user.is-logged-in()

res = is-user-logged-in("Hugh Mann")
//res :Result(Bool, String) =  is-user-logged-in("Hugh Mann")

match(res)
  Ok(is-logged-in = Bool)   -> log("Am I logged in: \(is-logged-in)?")
  Err(msg         = String) -> log("Error determining if logged in b/c: \(msg)")
```

### Pattern Match

### Limitations

### Examples



Notes:
* not-propogate unless explicit return?
* must handle in current context?
  * ie don't allow propogation and instead use more complex system?
* a way to append multiple contextual details to caller of caller?
  * like Haskell-style threading and Monad composing of State monad?
    * I think Swift does some of this? maybe Scala as well?
* combine w/ builtin functions like throw, catch, etc. and others?
  * look at Cone, Zig, Rust and how they combines these concepts?
  * or more useful to keep it very simple?


## Exception System
------------------------------------------------------------------------------------------------------------

Notes:
* An overall better idea to have an condition system?
  * or this a more complex and we have simple and complex ways to deal w/ error?
  * a compilation of: issue, resolution (proposed), solution
* what ways can we improve a condition system?
  * copy what makes sense from Zig?
* the concext in which the issue occurs has the most contextual details
  * ideally we can fix in place?
    * also means we don't have to manage memory up/down the stack and instead stays in place?
* 


## Lookup Table
------------------------------------------------------------------------------------------------------------


## Other Static Structures
------------------------------------------------------------------------------------------------------------

Notes:
* Reference
* Lookup Table
* Branch/Jump Table
* Optional/Result
  * of is there a better abstraction like Maybe? Either?, something better?
    * what type of pattern is being described?
  * differing from exception pattern as a means of handling simple issues?
    * have diff levels of handling? that which expects to fail and that which doesn't?
      * Result
        * Unknown if true?
      * Exception?
      * 
      * Panic
      * Reject Pogram
* Comptime and runtime mixed?
* Varargs? or this just a record w/ specific sugar?


## Selection
------------------------------------------------------------------------------------------------------------

### Range

Ranges can be generated over ordered data.
This extends to more than numbers to anything which has a comptime known ordering.

There are two syntaxes for ranges:
* inclusive
  * includes start and last index
  * eg
    ```
    1 .. 4  =>  1, 2, 3, 4 // or '..=' not conflict w/ dirs?
    b .. e  =>  b, c, d, e
    ```
* exclusive
  * includes start but not last index
  * eg
    ```
    1 ..< 4  =>  1, 2, 3
    b ..< e  =>  b, c, d
    ```

The default increment is the unit value.
An increment can also be specified.
This is especially useful for fractions and fixed point ranges:
```
1, 1.1 .. 1.5           =>  1, 1.1, 1.2, 1.3, 1.4, 1.5
1./20, 3./20 .. 15./20  =>  1./20, 3./20, 6./20, 9./20, 12./20, 15./20
```

TODO:
* others useful? any pattern the compiler can track?
  * can we determine most arithmetic?
* floating point?
* 

### Path

Relator path syntax is the simplest and most direct method of picking one-to-many names within a structure.

There are a few simple methods of selecting by path:
* by name
  * when names are present, they can be used to traverse data via a path
  * this works similar to structs/maps in other languages
* by comptime index
  * represents the syntax as initially defined in code
  * the index of the value in the record
  * index starts at 0
* by data index
  * just like comptime index but data indexes represent the current data
  * in later levels, data may change which means the same select may refer to different values
  * for this level, data and comptime indexes are the same

Consider a structure with multiple, nested relators:
```
o
  .a = 0
  .b
    :x = 1
    :y = 2
  .c = 3
    :z = 4
```

There are multiple ways to select `x` within `o`:
```
o.b:x   // by name
o.#1:#0 // by comptime index
o.1:0   // by data index
o.b:0   // by mixing them together
```

Similarly, there are multiple ways to select `x` and `y`:
```
o.b:(x, y)    // by name
o.#1:(#0, #1) // by comptime index
o.1:(0, 1)    // by data index
o.b:(0, #1)   // by mixing them together
```

Notes:
* others?
  * Leaves vs Branches
  * Multiple Names
  * Wildcard
  * Enumerable
* 

### Slice

Slices are views of existing data.
They do not implicitly copy the data that is slices.
This means they are very efficient and safer method of working with data, memory, etc.

Slices use the expression syntax with a range selector.
These work with both names and index.

Slices inherit the constraints of their target.
Slices are able to strengthen, but not loosen, constraints.

A simple example:
```
arr #mut = [a = 7, 8, 'my name' = 9, b = 10, 11, c = 12]

// slice by index
slice-0 = arr/(1 ..  4) // [8, 'my name' = 9, b = 10, 11]
slice-1 = arr/(1 ..< 4) // [8, 'my name' = 9, b = 10]

// slice by name
slice-2 = arr/('my name' ..  c) // ['my name' = 9, b = 10, 11, c = 12]
slice-3 = arr/('my name' ..< c) // ['my name' = 9, b = 10, 11]

// slice by meta
slice-4 = arr/(2         .. arr:end) // ['my name' = 9, b = 10, 11, c = 12]
slice-5 = arr/(arr:start .. 2)       // [a = 7, 8, 'my name' = 9]
slice-6 = arr/(arr:start .. arr:end) // [a = 7, 8, 'my name' = 9, b = 10, 11, c = 12]

// Mutate slice
slice-0/1 = -1

// Show that original array is updated
log(arr) // [a = 7, 8, 'my name' = -1, b = 10, 11, c = 12]

// Invalid
arr/(2 ..< 2)  // range cannot be zero
arr/(2 ..< 1)  // range-start must be < range-end
arr/(-1 ..< 1) // negative index
```


Notes:
* types of ranges
* data
  * reference data
  * copy data
  * other relationships w/ data?
    * transfer? borrow?
* invalid cases
* 


## Destructure
------------------------------------------------------------------------------------------------------------

Notes:
* should this be focuses on relators or shape? or both?
  * relators apply across shape?
* types:
  * array?
  * struct?
    * eg
      ```
      x      = { a = 1, b = 2, c = 3 }
      (d, e) = x/(a, c) ???
      (d, e) = x/(0, 2) ???
      (d, e) = x.(a, c) ???
      (d, e) = x.(0, 2) ???
      ```
  * optional/maybe?
  * exception?
  * relators and records?
  * type
    * or is this really comptime Fn input?
  * 


## Pattern Matching
------------------------------------------------------------------------------------------------------------

It is very common for a section of code to follow this pattern:
* decompose a complex structure into parts
* extract only the elements of a structure which are relevant
* determine all variants
* handle each variant in different ways

Pattern matching allows us to do all of this in a way often easier to understand than many conditionals.
When used appropriately, pattern matching can significantly improve the readability of code.

### Overview

Pattern matching follows the general syntactical form of:
```
match(elem-0, elem-1, ...)
  (cond-0-0, cond-0-1, ...) -> handler-0
  (cond-1-0, cond-1-1, ...) -> handler-1
  (cond-2-0, cond-2-1, ...) -> handler-2
  _                         -> handler-else
```
Notes:
* Each input of `match` are considered when matching
* Each child of `match` is an arm to test
* Each `cond` tests in order
  * If any `cond` names are emitted, they're passed to the next condition until finally reaching the handler
  * All names from each `cond` are passed down until reaching the end of the arm at their `handler`
* Conditions and handling are delimited by the syntactical sugar `->` (arrow)
  * The `->` (arrow) sugar is added to make the condition and handler clearly delimited.
  * `->` precedence is lower than `|` but higher than `\n`
* The *else* condition which covers all the remaining cases uses `_` (or `None`) to indicate there's nothing to test and all conditions will match
  * All cases must be handled.
    Either by all possible cases being provably handled or by using an *else* condition.

### Match on Type

Matching consideres each arm in order.
Even if some arms are more specific they are always tested in order.

Any names which are used in each condition are passed each each subsequent condition and handler.
All conditions must match for the handler to be matched.

```
Point = Struct
  x :U8 = 0
  y :U8 = 0
  z :U8 = 0

pt = Point(x = 1, y = 2, z = 3)

match(pt)
  Point(x = 0, y, z)     -> log("arm-0 matched in (\(x), \(y), \(z))")
  Point(x = 1, y, z)     -> log("arm-1 matched in (\(x), \(y), \(z))")
  Point(x = 1, y, z = 3) -> log("arm-2 matched in (\(x), \(y), \(z))")

// outputs log: "arm-1 matched in (1, 2, 3)"
```

### Match on Literal

Literals can be matched using the syntax of the literal itself.

Like other forms of matching, the names which are used are always emitted to the next.

```
// Array
arr = [1 .. 10]

match arr
  [x, _, _, y = 4, z, ...rest] -> ... // emit (x = 1, y = 4, z = 5, rest = [6, 7, 8, 9, 10])

// Record
rec = (a = 1 .. j = 10)

match rec
  (x = 1, _, _, y = 4, z, ...rest) -> .. // emit (x = 1, y = 4, z = 5, rest = (f = 6, g = 7, h = 8, i = 9, j = 10))

// Struct
str = {a = 1 .. j = 10}

match str
  {x = 1, _, _, y = 4, z, ...rest} -> ... // emit (x = 1, y = 4, z = 5, rest = {f = 6, g = 7, h = 8, i = 9, j = 10})
```

### Match on Relator


### Match on Condition

Multiple conditions can be tested.

Explicit 

Notes:
* multiple conditions
* introduce new names
* subsequence cond/handler use the names as they're passed down
* how to delimit multiple conditions
* shadowing?
* which variables are considered in match (only those passed or in scope?)

```
pt = Point(x = 1, y = 2, z = 3)
a  = 4
b  = 5
c  = 6

match(pt)
  Point(x = 0, y, z) | z == 3          -> ...
  Point(x = 1, y, z) | y == 3 | z == 4 -> ...
  Point(x = 1, y, z = 3)               -> ...
```


### Match on Expression


### Nested Match


### Match Function


### Match Scope

Notes:
* local scope?
  * local scope + specific names?
* other types of variables, meta names, etc.



Notes:
* multiple aspects
  * shape/structure
  * relator
* a generalization of case?


## Control Flow
------------------------------------------------------------------------------------------------------------

### `if` conditional

Simple conditional testing is handled by the builtin `if` conditional which:
* has sugar for multiple syntax forms
* is an expression and replaces the need for a ternary operator
* only tests `Bool` types (there aren't implicit casts)
* often uses comparison (`==, !=, <, >, <=, >=`) and logical operators (`and, or, not`)

Some examples of its usage:
```
x = 0
y = 1
z = 2

// These are equivalent
a =
  if
    x > 0 and y < 2
  then
    x
  else
    y

a = if (x > 0 and y < 2) then x else y

log a // 1

// These are equivalent
b =
  if
    x > 0 and y < 2
  then
    x

b = if (x > 0 and y < 2) then x

log b      // None         ?
log b:type // Optional(U8) ?
log b:type // U8 or None   ?

//    if
//      x
//      y
//      z
//    then
//      ...
//    else
//      ...
//    
//    if (x > 0)
//      log x
//    
//    if (x, y, z) then (...)
//    if (x, y, z) then (...) else (...)
//    
//    if (x and y and z) then (...)
//    if (x and y and z) then (...) else (...)
```

Notes:
* if, else-if, else
* if is expression
* if-let pattern?
* bind to relators
  * which relators are defined?
* operators
  * Literals: True, False
  * comparison: ==, !=, <, >, <=, >=
  * Boolean ops: and, or, not
    * mix w/ comparison args
      * eg `x > 0 and not y < 2`? or must be `(x > 0) and not (y < 2)`


### `loop`

Notes:
* bind to relators
  * which relators are defined?
  * eg `loop:it` or `loop/it`?
  * hold output, totals, intermediate variables in relators
    * that way can avoid common pattern of declaring totals?
* else
  * when never runs? or use after?
* before, body, after
* continue expression
  * used for incrementing?
* break w/ result
* labeled how?
  * how to differentiate from result?
    * `l = loop ...` and then later can use `l:result`?
* inline
* equivalent to `for`?
  * parts: init (introduce vars, run once), test (when to continue loop), increment (after body)
    * becomes:
      * before 'for'
      * test before body
      * after body 
    * ideas
      ```
      loop
      ```
* unify?
  * `do`
  * `for`
  * `while`


## Operators
------------------------------------------------------------------------------------------------------------

Instead of allowing arbitrary operators, there are a few builtin sets.
These sets have special rules to match most languages.

Developers are not allowed to create their own operators.
The primary reason is there's no good way to express intent with symbols.
The language is able to do so by enforcing a universal interpretation.

Only similar groups of operators can be mixed without parentheses.
Precedence is non-transitive and doesn't work between categories of operators.

### Invoke

Operators can also be a few forms:
* unary prefix: `-1`, `~0b1010`, `not True`
* binary infix: `3 + 4`, `True or False`
  * infix requires a space between the operands
* infix invoked like a fuction
  * this is sugar which distributes the infix operator over variable arguments
  * useful for building subexpressions with repeated operator
  * eg
    * `x + -(1, 2, 3)` becomes `x + (1 - 2 - 3)`
    * `a and or(b, c, d)` becomes `a and (b or c or d)`

### Arithmetic

These all work on `Numeric` types.
They follow the canonical mathematical precedence.

In order of precedence:
```
+1     // positive  (unary,  prefix)
-1     // negation  (unary,  prefix)
1 ** 2 // power     (binary, infix)
1 *  2 // multiply  (binary, infix)
1 /  2 // divide    (binary, infix)
1 +  2 // add       (binary, infix)
1 -  2 // substract (binary, infix)
1 %  2 // remainder division (binary, infix)
1 %% 2 // modulus floor? (binary, infix)
```

TODO:
* overflow behavior
  * overflow
  * wrapping
  * saturation
  * error flag

### Logic

These only operate on `Bool` values.

//These operate on many different types of values.
//Most often, they're used with `Bool` types.
//However they're also used for union and disjunction types.

They follow the canonical logic precedence.

In order of precedence:
```
not False // Unary negation
a and b   // Conjunction
a or  b   // Disjunction
```

Which means this parses like:
```
a and not b or c and not d

// equivalent to:
(a and (not b)) or (c and (not d))
```

### Comparison

These work on any values which can be compared and ordered.
They always result in a `Bool` type.

In order of precedence:
```
a == b // equivalent by value
a != b // not equivalent by value
a <  b // less than
a >  b // greater than
a <= b // less than or equal equivalent value
a >= b // greater than or equal equivalent value
```

Since comparison operators can only be used with one another, we allow some sugar for chaining multiple conditions.
These only work for comparison operators (`<`, `>`, `<=`, `>=`) like:
```
a <  b <  c  =>  (a < b)  and (b < c)
a >  b >  c  =>  (a > b)  and (b > c)
a <= b <= c  =>  (a <= b) and (b <= c)
a >= b >= c  =>  (a >= b) and (b >= c)
a <  b >= c  =>  (a < b)  and (b >= c)
a >  b <= c  =>  (a > b)  and (b <= c)
```



Q:
* others?
  ```
  a is   b // identity ??? this may be equivalent as: a:ref == b:ref ?
  a isnt b // not identity ??? this may be equivalent as: a:ref != b:ref ?
  a eq   b // ???
  a neq  b // ???
  a id   b // ???
  a nid  b // ???
  ```

### Bitwise

In order of precedence:
```
~  // Bitwise unary negation
>> // Bit shift right
<< // Bit shift left
&  // Bitwise and
|  // Bitwise or
^  // Bitwise xor
```

Notes:
* Should these be specific extensions by import and the pattern means bitwise on import?
  * b/c not generally useful?

### Equality Testing?


### Mixed Operators and Functions

Operators have a relative precedence.
This means within a group of operators they have precedence within the group.
There is no precedence for operators across groups.
It is also often not meaningful to mix operators without parentheses due to not having implicit casts (for example this is not meaningful `1 + (2 == 3)`).

For practicality reasons, there is a global precedence for the main classifications.
This was chosed as there's on a few rules to remember and it generally follows what developers expect.

The precedence across classifications is (high to low):
* Relator Path (`a/b:c.d`)
* Unary Operator (`-a, not b`)
* Infix Operator (`a + b, a or b, a <= b`)
* Function Invocation (`sum a + b`)
* Assign (`x = a`)

This doesn't mean that different types of operators can still be logically mixed.
For example, this isn't meaningful:
```
a < b or not c == d
```
This mixes multiple classifications of operators of comparison and boolean.

Instead this would have to be reworked using parentheses:
```
(a < b) or not (c == d)
```

Notes:
* what can, if anything, be mixed?
  * function invocation? assign? relator?
* how to handle association?
* others in Zig:
  ```
  a +% b,  a +%= b   // wrapping add
  a +| b,  a +|= b   // saturation add
  a -% b,  a -%= b   // wrapping minus
  a -| b,  a -|= b   // saturation minus
  -%a                // saturation negation
  a *% b,  a *%= b   // wrapping multiply
  a *| b,  a *|= b   // saturation multiply
  a <<| b, a <<|= b  // saturate shift left
  a ++ b             // array concat (eg "ab" ++ "cd")
  a ** b             // repeating pattern (eg "ab" ** 3)
  ```



## Lambdas
------------------------------------------------------------------------------------------------------------

Lambdas are similar to simple functions without names.

Lambdas require that:
* there are two records
  * the first is interpreted as the list of input arguments
  * the second is interpreted as the block to execute
* the output type is inferred as the last statement of the block

### Syntax

The `->` (arrow) sugar is added to make the input and body clearly delimited.
`->` precedence is lower than `|` but higher than `\n` like:
```
(x :Int = 0, y :Int = 1) -> (z = abs(x - y), z += 1)
```

It is allowed to put the body on a indented newline after the arrow.
It will be interpret the indented block as the block to execute.
This is equivalent to the previous example:
```
(x :Int = 0, y :Int = 1) ->
  z = abs(x - y)
  z += 1
```

When this translates to IR it is much more explicit:
```
// Notice everything is named in IR to be explicit, even Lambdas
lambda-0 = Lambda()
  :in
    x = 0
      :type = Int
    y = 1
      :type = Int
  :body
    z :Int = abs(lambda-0:in/x - lambda-0:in/y)
    z += 1
    lambda-0:out/0 = z
```

### Usage

Lambdas are often used in a few cases like:
* a inline function literal
* a handler for pattern matching
* a step in a pipeline

It is possible, although rare, to execute directly.
Lambdas use the same calling syntax as functions:
```
((x :Int = 0, y :Int = 1) -> (z = abs(x - y), z += 1))(2, 5) // 4
((x :Int = 0, y :Int = 1) -> (z = abs(x - y), z += 1))()     // 2
```

### Currying? Static Function Binding?

Q:
* only allowable in zero-cost?
  * either const or zero cost or copied?
    * how does it deal w/ reference types b/c would this cause to be mutable workaround?
* pass control to stack to the curried fn?
  * then the next call is actually appended to the stack instead of popped?
    * like an implicit continuation?
    * is this a general pattern for generators/closures?
* 

### Closure

```
// defn
init = Fn (start :Int)
  inc = Fn ()
    start += 1

// usage
do-inc = init(0)
do-inc()
do-inc()
do-inc()

becomes:

??? want for stack to only increase like a continuation ???
- put the 'start' on the stack and can use from that point on for rest of stack?
```


## Functions
------------------------------------------------------------------------------------------------------------

Functions extend Lambdas in a few ways:
* they require assigning a name
* three records are required
  * the first is interpreted as the list of input arguments
  * the optional middle is interpreted as the list of output arguments
    * when not present, the last statement of the executed block is inferred
  * the last is interpreted as the block to execute

### Definition

Different abstractions define different relators.
The `Type` of the abstraction determines how each relator is interpreted.
Functions are a special type of container which does this.
It interprets its relators in order to describe input, output, execution, and others.

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
  * Has relators (which act like keywords) to make development more terse of:
    * `Fn:body/return` for explicit early return

A simple example which are all equivalent:
```
// Infer output
sum = Fn (x :U8 = 1, y :U8 = 2)
  x + y

// Explicit output type
sum = Fn (x :U8 = 1, y :U8 = 2) -> U8
  x + y

// Multiline using relators
sum = Fn
  :in  = (x :U8 = 1, y :U8 = 2)
  :out = U8
  :body
    x + y

// Fully explicit IR
sum = Fn()
  :in
    x = 1
      :type = U8
    y = 2
      :type = U8
  :out
    U8
  :body
    sum:out/0 = sum:in/x + sum:in/y
```
Notice that:
* The sugar arrow `->` delimits the input and output relators
  * This is only valid inline
* The last expression of `Fn:body` is the return value
* The sugar maps to a very explicit form in the IR

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
_ = sum(1, 2)

// Illegal
sum(1, 2)                   // can't implicitly ignore return value, must be explicit
z       = sum(x = False)    // 'x' type doesn't match 'sum' definition
z       = sum(1, 2, 3)      // too many args
z       = sum(x = 1, x = 2) // 'x' isn't unique
z       = sum(z = 3)        // 'z' isn't defined in 'sum'
z :Bool = sum(1, 2)         // 'z' type doesn't match 'sum' definition
```

### Using `in` and `out`

A developer can use the `:in` and `:out` relators.
Often this is more explicit than constructing arbitrary names to represent input and output.
It also helps to dissambiguate when nesting functions.

Consider a extending the sum function which manually detects overflow:
```
// Use names :in and :out
sum = Fn (x :U8 = 1, y :U8 = 2) -> (z :U8, is-overflow :Bool)
  tmp-sum :U16 = x + y
  is-overflow  = tmp-sum > U8:max

  if (!is-overflow)
    z = tmp-sum

// A equivalent form using Fn:out relator
sum = Fn (x :U8 = 1, y :U8 = 2) -> (U8, Bool)
  tmp-sum :U16 = x + y
  sum:out/1    = tmp-sum > U8:max

  if (!is-overflow)
    sum:out/0 = tmp-sum

// A equivalent form using last expression
sum = Fn (x :U8 = 1, y :U8 = 2) -> (U8, Bool)
  tmp-sum :U16 = x + y
  is-overflow  = tmp-sum > U8:max

  (tmp-sum, is-overflow)

// Usage
(z, _)      = sum(1, 2)      // Ignore error flag (no overflow detected)
(z, is-err) = sum(1, 2)      // Store error flag (no overflow detected)
(z, is-err) = sum(1, U8:max) // Store error flag (overflow detected)
```
Notice that `out` names are:
* Used directly to for locally scoped values as well as output.
  * This makes it easier to avoid adding arbitrary names.
* Destructured just like any other record.

### Nesting

Code that is more local, linear, and isolated is often easier to reason about.
To that end, functions can nest arbitrarily deep so they can be as local as possible.

A simple example:
```
average-squares = Fn (widths :[]U8, heights :[]U8, depths :[]U8) -> (U8, U8, U8)

  // Declare inner function to sum over array
  average = Fn (arr :[]U8) -> U8
    tmp :U8

    each(arr)
      tmp += arr:it

    tmp

  average(widths), average(heights), average(depths)
```

This does not create closures as it would require allocation hidden from the developer.
Instead, this desugars to two functions where the nested functions are actually defined in the same scope and just generated on behalf of the developer.
In cases where a closure would be created a developer must opt-in to an abstraction which can allocate them.

### Variable Lifetimes, Locations, and Scopes

Variables can interface in more complex ways than many other languages.
The intent is to provide more zero-cost options in order to best model your solution.

There are multiple targets where variables can be defined respective to a `Fn` (in order):
* `Fn:enter/before`
  * immediately before control is passed to invoked function
* `Fn:enter/fst`
  * immediately upon entering the invoked function
  * If a recursive function, this is only invoked once on first invocation
* `Fn:enter`
  * immediately upon entering the invoked function this is run
  * If a recursive function, this is invoked each time
* `Fn:exit`
  * right before control is returned from the invoked function
  * If a recursive function, this is invoked each time
* `Fn:exit/final`
  * right before control is returned from the invoked function
  * If a recursive function, this is only invoked once on the last return
* `Fn:exit/after`
  * immediately after returning contol to caller
* others?
  * before and after caller (1 additional stack frame)
    * this for propogation of values to survive until no longer needed?
  * auto variable liftimes?
    * propogate across multiple calls when last across all lifetimes?

Similarly there are multiple storage locations to aid in persistance across invocations, threads, modules, etc.
Like most else they're associated to specific relators:
* `Fn:global`
  * Like C-style `static` where variable persists across all invocations in `data` segment
* `Fn:thread-local`
  * Exactly like `Fn:global` but is also thread-local so this can be shared across multiple threads
* others?
  * `Fn:branch`
    * stored while entire branch is still active and not reaped? can be accessed by any decendent?
    * implicitly is thread-local b/c uses the thread-specific stack
  * `Fn:index`
    * eg index of cpu or enum?
  * `Fn:key`
    * eg program phase? enum?

### Additional Relators

Notes:
* first-enter (w/n function)
* final-exit (w/n function)
* before-enter/when-called (before pass to caller)
* after-end/when-returned (after return to caller)

### Tail Recursion

### Modifiers

Functions have modifiers like:
* `#inline`: suggestion to compiler to inline where possible
* `#must-inline`: must inline, else comptime exception
* `#stackless`: ???
* `#recursive`: ???


### Reusing Function Relators

TODO:
* hook
* mix w/ meta?
* stack specific propogating values
  * in stack and propogate to children/called
  * safe?
* branch-specific comptime values and memory
* should we describe closure and such here or wait until we move to CLI or others?
* 

### Constrained Closure


### Dropping Parentheses

Notes:
* relates to association and precedence
* right associative
* only on the outermost functions and not mixed fixity?
  * eg `f g h x => f(g(h(x)))` is fine but `f x + g y + z` is not?
* 

Notes:
* rest and varargs? or this a more generic concept?


## Protocol
------------------------------------------------------------------------------------------------------------

Protocols are the main means of abstraction.
They describe nearly arbitrary constraints and patterns.

Protocols have several components which work in coordination:
* Definition
  * Its requirements, fields, types, kinds, etc.
* Implementation
  * Description of how a specific type (or even protocol) conform to the protocol
  * The implemention of the protocol and how it should be applied
* Validation
  * The algorithm which validates the implementation
  * Most commonly this is covariant and contravariant restrictions
* Usage (integration better name?)
  * How the protocol interacts with other constructs like memory, arrays, functions, etc.

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

Implements(Dog, Speak)
  .speak = log("bark")

Implements(Cat, Speak)
  .speak = log("meow")

Implements(Duck, Speak)
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

Implements(d = Dog, proto = Speak)
  .speak = log(d.message)

Implements(c = Cat, proto = Speak)
  .speak = log(c.message)

Implements(d = Duck, proto = Speak)
  .speak = log(d.message)
```

### Protocols of Protocols

### Prerequisites

### Added Constraints

Notes:
* Move from `Implements` to `Protocol:Implements`?
* Use additional relators to define different types of constraints
  * will need something b/c need to use free vars, fns, data to do meta generation?
  * eg
    * `Protocol:has`   = has specific fields
    * `Protocol:like`  = matches the structure in a specific way while respecting other relators and format
    * `Protocol:match` = 
    * `Protocol:mimic` = 
    * `Protocol:rule`  = introduce a rule
* Diff ways to implement protocols (cost and resolutions)
  * eg builting Numeric types
* 

### Limitations

As mentioned in the intro, Tozen doesn't have a runtime by default.
This prevents the developer from writing non-optimal code without their knowledge.
A developer must opt-into costs.

Depending on their use, protocols can conflict with this requirement.
Here are some common examples which have hidden costs:
* ...protocol in list...
* ...protocol in conditional...
* ...protocol to indirect memory? or this just 2 costs of indirection + protocol-in-list?...
* ...others?...


## Exception
------------------------------------------------------------------------------------------------------------

Notes:
* Exceptions should be like guards and allow to pass else exit the entire block
  * else we continue to stack more and more nested blocks and make harder to read
* Ideally many excpetions can be eliminated at comptime and known they will be handled
* Alternative construct
  * Issue
    * Shape of Issue
  * Solutions
    * Proposed Solutions (from issue)
      * Easier to know from w/n sometimes?
    * External Solutions
  * Resolution
    * Pick solution
    * Add implementation details of solution which guarantees handles all issues
* Rust-like Result
  * pattern match on simple results and issues
  * not ideal for more complex?
* Categories of exceptions
  * factors
    * knowable
    * recoverable
      * handlable
    * 
* goals
  * avoid propogating exceptions
    * better to declare at a specific level and propogate decision of resolution up
      * also solution can be added up
  * recover a boundries of unit of work
    * not changed or recoverable
  * don't affect other boundries
    * eg server API request should not affect another API request
  * 
* different categories of responses
* diff parts of an exception
  * declaration: what error can occur
  * proposals: how diff levels of code which use exceptional code could handle
  * selection: selection and uses of proposal(s)
  * resolution: how proposal(s) are implemented and details of resolving
  * others?
    * implementation details?
    * effects?
* how to codify Protocol depending on implementation will have a risk?
  * eg `Integral / Integral` is not able to error if `Integral = Whole` but will if `Integral = Int` (or anything which constains zero
  * how to make this understandable?
* more about issues and resolution
  * if we want to ignore the issue then it should be fully erased?
* 


## Module
------------------------------------------------------------------------------------------------------------


## Memory
------------------------------------------------------------------------------------------------------------

Memory management is only relevant for some build targets.
When targeting JavaScript VM, manual memory management isn't very important.
When targeting native, it is essential.



Notes:
* Often handled for you depending on memory layout, access patterns, and compositions
* Compatibility compiler directives to make compatible w/ C API, packed, and custom layouts (eg data)
* Default/Global allocator?
  * eg JavaScript allocator is to JavaScript VM GC
  * eg HTML allocator is to HTML static analysis or None
