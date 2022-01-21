TODO:
* memory issues
  * ADT/Tagged Union issues
    * single values are fine; but not when stored in aggregate
    * additioanlly ADT have extra contextual info which must be copied along?
* exceptions
  * all issues should be declared and must be handled
    * this doesn't mean must be an exception as there's other ways to handle
    * every edge-case must be declared; no implicit cases
  * some are exceptions which can't be propogated and some can
    * if not propogated, they must be handled locally
* builder
  * eg range
  * emit to comptime known? and then available to analysis?
* const propogation
  * from top -> inner?
* usage patterns
  * var and others
* different storage locations, lifetimes, and other points in code
* knowable checker and abstractions
  * separate and be clear
* static code asserts and patterns
  * so dev doesn't have to manually check
  * eg shadow
* clear mutation from caller syntax
  * this not always work b/c sometimes conditionally mutate from caller?
* propogate stack vars from called to caller
  * can determine when earliest point to reap
* function args RO?
  * Ptr/Ref will be copied but content not?
* hidden terms are modeled like local variables in comptime block?
  * they're expanded before the declaration which is the final statement?
* safe points?
  * jump back and erase progress?
* comptime tracking
  * eg know min/max of counts, number of time, etc. so can determine max value
* comptime fields
  * eg when reading a literal all the fields the compiler provides for you?
    ```
    x = 432.68

    // compiler fields are:
    x:type         = DecimalLiteral
    x:is-signed    = false
    x:integral-len = 3
    x:decimal-len  = 2
    ```
* allow dev to extend their own static domain
  * eg security constructs w/ specific rulesets and usage
* how does the order of evalution and bind:
  ```
  // ordering ?
  // - invoke
  // - relator
  // - bind
  result = f()
    .x = 1
    :y = 2
    /z = 3
  ```
  * how does this jive w/ sugar constructor?
* consider renaming this section to known or compiler known
  * means these are special abstractions that are:
    * builtin to language
    * zero cost
    * erasable
    * valid
    * defined
    * total
  * imposible to make programs which compiler can't fully reason about
* builtin types
  * related to flattening so the compiler knows everything?
  * 
* meaning of erased
  * for us, it means the abstractions are fully (or nearly so) eliminated by runtime
  * a developer is free to use them w/o fear of any cost
  * the only added cost is comptime speed but that will be ameliorated by smart caching?
* static placeholders
  * can be made to be total over entire structure
  * similar to templating?
  * similar to wholes?
* exhaustiveness and total checking
  * over structure b/c no functions yet?
    * or just static transformations?
  * eg regex or transforms
  * eg build up in steps into a static structure
  * irrefutable? other options?
* matching
  * related to irrefutable
  * pattern-matching and much more?
    * or should expand as gets more complex?
* goals for tools
  * flat
  * linear
  * build-up-incrementally
  * zero/once usage
  * comptime process/expand/select/knowledge
  * 
* templates
  * look at Pug?
* replacable patterns
  * eg struct-like
* substitution (basic variables? immutable? scopeless?)
* total transformations (not functions though)
  * exhaustive
  * irrefutable
* types and constraints
* schema, protocol, traits, interfaces, etc.?
* contexts and knowns in contexts
  * contexts have man rules? not just knowns?
* refinement types & constrained dependent types?
* syntax and patterns
* defaulting (or this in core?)
* query
* basic types and literals
  * all statically sized and dynamic later?
* modifiers?
  * const, mut, ...
  * move more to as local as possible?
  * differences between:
    ```
    #x = ...
    x #= ...
    x = #(...)
    ```
* syntax pattern
  * simple expansion and transformations?
  * simple comptime?
* schema separate from types, constraints, etc.?
  * schema implies different things
    * schema of structure (names, parent-child, etc.)
      * this also what we would also use 'syntax patterns' for? common, repeated structure?
    * schema of constraints (types, refinements, etc.)
  * eg InternetObject
    ```
    name, age:{int, min:20 }, address: {street, city, state}, active?:bool, tags?:[string]
    ---
    ~ Spiderman, 25, {Bond Street, New York, NY}, T, [agile, swift]
    ~ Ironman, 48, {Malibu Point 10880, Malibu, CA}
    ```
* orderings
  * and other known relationships?
    * enumerations, ordering, partial-order, etc.
* static joins and substitution
  * eg `"a" "b" "c"` ?
* compount types (finite)
  * literals
  * builtins
    * array
    * map
    * enumeration
    * ADT
    * range
    * 
    * 
* finite, static, immutable, erasable types
  * List-like
    * homogeneous
    * names are alias to value (zero-cost)
    * when no type prefix, desugar to Record
    * syntax is `[]` or `#[]`
  * Map-like
    * heterogeneous
      * type is inferred
    * names are alias to value (zero-cost)
    * when no type prefix, desugar to Record
    * names and values are inferred like `:`
    * syntax is `{}` or `#{}`
* should we allow converstions?
  * only among the builtins but not user?
    * what issues this cause if just builtings or universal?
* Tagged Union
* Finite State Machines
* Functions
  * Total functions
  * Builtin functions w/ guarantees
* Finalizing operator?
  * or this more valuable for mutation?
* 

Q:
* Should we have a target environment default types (eg Int, UInt, etc.)? and additionally have well-defined?
  * Also have other optimal builtins based on what the compiler determines from the target environement?
* 

## Reserved and Builtin
### Primitives
    * Integral
      * `ISize` works like `intptr_t`
      * `USize` works like `uintptr_t`
    * C ABI compatibility
      * `C-Char` is `char`
      * `C-Short` is `short`
      * `C-UShort` is `unsigned short`
      * `C-Int` is `int`
      * `C-UInt` is `unsigned int`
      * `C-Long` is `long`
      * `C-ULong` is `unsigned long`
      * `C-LongLong` is `long long`
      * `C-ULongLong` is `unsigned long long`
      * `C-LongDouble` is `long double`
      * `C-Void` is `void`
    * Target ABI compatibility
      * 

## Compound Literals

Notes:
* `[]` and `{}`
* finite, static, immutable, erasable types
  * List-like
    * homogeneous
    * names are alias to value (zero-cost)
    * when no type prefix, desugar to Record
    * syntax is `[]` or `#[]`
  * Map-like
    * heterogeneous
      * type is inferred
    * names are alias to value (zero-cost)
    * when no type prefix, desugar to Record
    * names and values are inferred like `:`
    * syntax is `{}` or `#{}`
* Defer to later?
  * placeholder until usage? `Undefined`?

### Access

```
x = [...]
x .[...]
x.[...]
x [...]
x[...]
```

```
// Can access via name or index just like record
x.[a, 3] // meaningful vs x.(a, 3)? (1, 3)

// Can access via name or index just like record
x.{a, 3} // meaningful vs x.(a, 3)? (1, 3)
```

```
x = {
  a b = c d <desugars-to> 'a b' = "c d"?
```

## Query
------------------------------------------------------------------------------------------------------------

## TODO - Refinement Types (or later b/c requires predicates? or should introduce predicates here?)
------------------------------------------------------------------------------------------------------------

Refinement types are special in that they're an extension beyond Types.
Unlike `Dependent Types` they don't require the developers to provide proofs and thus less powerful.
The benefit is they're nearly free and automattic for the developer to use.

## Types
### Notation

These are all equivalent based on the needs of the developer (from more to less terse):
```
x:Int = 1

x :Int = 1

x = 1
  :Int

x
  = 1
  :Int
```

Notice this is a natural extension of the Tozen data-oriented syntax just applied to the context of variable definition.

This is a common pattern of:
* The Tozen data-oriented syntax defines the notation
* The notation is used in some domain (in this case variable definition)
* The domain gives meaning and interpretation to the notation (the semantics)

### Syntax for Types and Structure

One novel aspect of Tozen is that it's idiomatic to to use syntax directly where other languages will often use data.

For example, if we need to model a few staff members with a bit of structure:
```
staff =
  - Janus Jannson
    CTO
    36
    36581
  - Marky Mark
    Production Assistant
    26
    76504
  - Emily Esquire
    Producer
    46
    43864

this is equivalent to:

staff =
  Janus Jannson, CTO,                  36, 36581
  Marky Mark,    Production Assistant, 26, 76504
  Emily Esquire, Producer,             46, 43864
```

The compiler views `staff` as:
* a `Sequence` of 3 elements
* each element is a `Sequence` of 4 elements

Tozen doesn't require that these are made into any form of concrete data.
Instead, the developer can use them as they need in multiple contexts.
Often the contexts they're used in will make them concrete.
For example, a JSON context or a DB connection context would use `staff` in different ways.

This has a close relationship to comptime which will be discussed in other documentation.

## Units
------------------------------------------------------------------------------------------------------------


## Simple User Types

Notes:
* build common types
  * static array-like (like C/Zig array)
  * static struct-like (like C/Zig struct)
  * static map-like (like C/Zig struct? but w/ more syntax requirements?)
  * static enum-like
  * static tagged union-like
  * static pointers/ref?
* related utilities
  * pattern matching?

### Type

```
Arr-Of-U8 = Alias(Array(U8))          ?
a         = Arr-Of-U8():init(3, 4, 5) ?
```

Sugar:
* constructors are always Type-specific sugar?
  ```
  U8(7)

  Array(U8)(1, 2, 3) =>
    Array()
      :init      = (1, 2, 3)
      :elem-type = U8

  Array(U8)[1, 2, 3]   =>
  Array(U8)([1, 2, 3]) =>
    Array()
      :init      = [1, 2, 3] => (1, 2, 3)
      :elem-type = U8

  Map(U8){a = 1, b = 2, c = 3}   =>
  Map(U8)({a = 1, b = 2, c = 3}) =>
    Map()
      :init      = {a = 1, b = 2, c = 3}
      :elem-type = U8

  Map(U8)(a = 1, b = 2, c = 3)   =>
    Map()
      :init      = (a = 1, b = 2, c = 3)
      :elem-type = U8
  ```
* 

### Array-like

Arrays respect the following relator paths:
* `:init` vs using constructor function?
  * or constructor should use sugar/shorthand?
    * eg `Array():init(3, 4, 5)` vs `Array(5 /* capacity */, U16 /* elem-type */):init(3, 4, 5)` vs `Array(3, 4, 5)`?
    * or this related to irrefutable patterns?

```
Array(1, 2, 3)
  :capacity  = 5
  :elem-type = U16
```

Notes:
* need way to differentiate between input to `Array` vs type of `Array`
* model nested arrays? how to read clearly?
  * explicit syntax here? builder/sugar in next?
    * sugar like `Array([3, U8][4, I8])` or something?
* consider things like slices?
  * for future layers?
* need multiple details like:
  * element type (even when complex like ptr)
    * `Array(Int)`
    * `Array(Ref(Int))`
  * number of elements
    * `Array(.capacity = 3, .len = 0)`
  * literal input
    * `Array([1, 2, 3])`
  * mix of many
    * `arr = Array(:type = Ref(Int), .capacity = 3, .len = 0)`
    * ```
      arr = Array()
        :type = Ref(Int)
        .capacity = 3
        .len = 0
      ```
    * ```
      arr = Array([1, 2, 3])
        :type = Ref(Int)
        .capacity = 3
        .len = 0
      ```

## Pattern Matching
------------------------------------------------------------------------------------------------------------

Pattern matching require irrefutable patterns.

```
type IntOrString =
   | Int i32
   | String string

match Int 7
| Int 3 -> print "Found 3!"
| Int n if n < 10 -> print "Found a small Int!"
| String "hello" -> print "Found a greeting!"
| value -> print "Found something else: $value"
```

## Simple Protocol
------------------------------------------------------------------------------------------------------------

### Basics

* other
  * Algo which verifies conforming to the protocol
  * Implement abstraction
    * Replacement? Solutions? Subst? Compatibility?
    * This is how once it conforms how do we implement the abstraction?
      * eg dynamic dispatch, vtable, indirection pointer
      * eg Cat, Dog, Person all implement Mammal Protocol. How do we:
        * have a function which allows Mammal? how is it implemented?
        * have a array which allows Mammal? how is it implemented?
        * have a variable which allows Mammal? how is it implemented?
          * eg `m :Mammal = rand-bool() ? Cat() : Dog()`
    * This is also about how other structures utilize and implement Protocols.
      * Each of these decisions have costs, memory patterns, etc. which are not always transparent to dev.
  * Costs?

### Compatibilty

Tozen allows for abstractions which are more dynamic; it just doesn't allow you to do so without opting-in to the additional costs.
At this level, we are only describing tools which the compiler can statically reason about.
For this reason, we will be moving some tools (eg dynamic dispatch, mixed memory layout) to the meta layer.



* and costs
  * mixed storage and access causes issues w/ memory layout and access
    * eg union has diff costs vs pointer indirect vs ...
  * memory
    * single size vs mixed (eg proto)
    * single layout vs mixed (eg union)
    * levels
      * SOA, AOS, units, ...
      * exactly the same size and layout (offsets)
      * same size w/ diff layout (eg union)
      * diff layout w/ diff size (eg indirect pointer w/ common fields)
      * indirection (eg memory lives elsewhere)
      * others?
  * code
    * static vs dynamic dispatch
  * non-determinism
    * eg overloading in unexpected way?
* 
* 

## Simple Functions
------------------------------------------------------------------------------------------------------------

### Constraints

* Dhall-like `forall`
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

## Free Structure
------------------------------------------------------------------------------------------------------------

Relators are used in many contexts.

In some contexts specific relator paths have been ascribed semantic meaning.
This is how many types define, describe, and are queried.

When not, relators can be used to add structure significant to the developer.
Since there isn't a semantic meaning attached to them they're interpreted just as they are.
Nested structure are interpreted as symbols and have no cost.

### Simple Example

```
main-colors =
  'red'
  'green'
  'blue'

highlight-colors =
  'white'
  'black'
  'grey'
  'yellow'


```

Notes:
* Relators allow for structure/namespacing w/o cost
* They are a literal structure interpreted data as-is (just like lower level)
* Users (or functions) interpret over relators to define meaning
  * Without a specific context, the paths are interpreted just as immutable symbols
