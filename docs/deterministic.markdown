## Relator
------------------------------------------------------------------------------------------------------------

Since Elder syntax can be used for various use-cases (from serializing data like JSON to imperative programming like C and more) it must be able to represent many ways to define, format, and structure data.

For example, consider other languages which are often use hard coded syntax:
* `C++`
  * accessing a `Namespace` uses `::` syntax like `my_namespace::my_val`
  * accessing a `Struct` field uses `.` syntax like `my_stuct.field` or `my_struct.field = 1`
* `Clojure`
  * accessing a `Namespace` uses `/` syntax like `my_namespace/my_val`
  * interop with `Java` or `JavaScript` uses `.` syntax like `(.toUpperCase "jane")`
* `XML`
  * accessing a child element uses `/` syntax like `car/wheel/rim`
  * accessing a attribute uses `.` syntax like `car.weight`

Elder syntax takes a different approach by allowing the developer to create their own syntax in a very constrainted way.
Customizing the syntax allows developers to add structure to the syntax without the need of macros.

To accomplish this, we introduce a new syntax we've termed a ***Relator***.
Specifically a ***Relator*** is defined as a syntactical tool which describes the ***relation*** between a ***object*** and a ***descriptor***.

Functionally it works much like a **namespace** but:
* characters can only be specific ASCII punctuation
* identifier characters and relator characters must not overlap b/c relators are used as delimiters between identifiers (like `a/b/c` where `/` is a relator)
* is more general concept than a namespace and can be used to model more patterns
* can be syntactically used in more ways
* it is a tree node in order to match layout of code to structure of data
* If multiple relators have the same syntax, the longest match wins
  * For example, if both `:` and `::` are defined `::` will always win
  * This also means multiple characters can be combined to make new relators.
    * For example, if `:`, `.`, `/` are all relators then `:./` would form a new relator distinct from the others.

Although developers can define their own relators, there are 3 relators which have a universal meaning:
* `/` the child relator for parent-child relation like:
  * for HTML, a DOM child element
  * for namespaces, a child namespace
  * for files, a directory to it's contents
* `.` the dot relator for object-property relation like:
  * for OOP (and many other) languages, accessing the fields of a instance
  * for HTML, accessing a attribute of a DOM element
* `:` the meta relator for data-metadata relation
  * used to represent metadata of any data (and since it's homoiconic it can target almost anything)
  * does the work of describing concepts like types, constraints, traits, interfaces, rules, etc.

### Literal Structure

In order to model data without additional syntactical noise, data can be written as literal trees using relators.

To keep the examples simple, I'll focus on the object-property relation `.` syntax.

With a few examples the rules become obvious:
```
obj
  .a = 0
  .b = 1
  .c = 2
```

and is equivalent to when grouped under a relator node:
```
obj
  .
    a = 0
    b = 1
    c = 2
```

and is also equivalent when adding the relator to the parent:
```
obj.
  a = 0
  b = 1
  c = 2
```

This flexibility is allowed b/c there are many use-cases and we should allow the developer to model the structure in the most applicable way.

In general, developers should prefer the structure which best models the data layout as directly as possible.

### Relator-Record Equivalence

```
o = 4
  .a = 0
  .b = 1
  x  = 2
  y  = 3

// equivalent to

o.(a = 0, b = 1)/(x = 2, y = 3) = 4
```


## Relator
------------------------------------------------------------------------------------------------------------

### Path

One aspect that's unique to relators is that they're used to create paths in order to:
* define data
* abstacts over concepts like file and network paths
* traverse data structure

Consider a slightly more complex example which uses both the `.` (object-property) and `:` (data-metadata) relators:
```
o
  .a = 0
  .b
    :x = 1
    :y = 2
    :z = 3
  .c = 4
    :t = 4
    :u = 6
```

Using paths we can access and define this data in multiple ways.
Syntactically paths acts more like other languages and uses the relator as a delimiter.

For example, to select `x` in `o` we write:
```
o.b:x // Selecting a value will return it's result
```

It's also useful in when paired with equals syntax to assign:
```
o.b:x = 7
```

### Select

A path describes a location.
A selection further refines a path by describing the names to use at that location.

Continuing the example from above we can select multiple values under `o`:
```
// Path is `o.b:` while selection of `x, z` picks only those names
o.b:(x, z)
```

In a similar way we can assign values just like before:
```
// Assign both names `x, z` to values
o.b:(x, z) = (7, 8)

// This translates to:
(o.b:x, o.b:z) = (7, 8)
```

### Multiple Paths

Paths and selection are not limited to the the same parent.

The trick of doing this consistently is that when multiple selections across paths are used it creates a single, flattened sequence.

Continuing the example from above we can select multiple values under `o` but different paths:
```
// Results in a single, flattened sequence
o.b:(x, z), o.c:(t, u)

This translates to:
o.b:x, o.b:z, o.c:t, o.c:u
```

In a similar way we can assign values just like before:
```
(o.b:(x, z), o.c:(t, u)) = (7, 8, 9, 10)

// This translates to:
(o.b:x, o.b:z, o.c:t, o.c:u) = (7, 8, 9, 10)
```



## Issues, Edge-Cases, Ambiguities, and Quirks
------------------------------------------------------------------------------------------------------------

### Multiple Paths

* Steps:
  * merge/join/flatten
  * expand
* Works when unambiguous
  * either is last-element assign OR entire assign so must be clear?
    ```
    x.(a, b), y.(c, d) = (1, 2, 3, 4)
                  vs
    (x.(a, b), y.(c, d)) = (1, 2, 3, 4)
    ```

```
x.(a, b) = (1, 2)
x.(a, b), y.(c, d) = (1, 2, 3, 4)
(x.(a, b), y.(c, d)) = (1, 2, 3, 4)

x.(a, b.(t, u, v), c) = 1..5
```

* why `x.(a, b) = (1, 2)` vs `(x.(a, b), y.(c, d)) = (1, 2, 3, 4)`?
  * shouldn't they act differently b/c first preserves parens while second drops them?
    * or select is across each item? always coerced to a flat listing?
      * eg `(a, b, (c, d, (e, f))) =` `=>` `(a, b, c, d, e, f)` when L-hand?


### Relator Orthogonal Syntax

Relator syntax has two primary purposes: selecting names or defining structure.
Selecting names uses the `Path + Select` pattern to pick names at a location (which we term *selection mode*).
Defining structure uses the `Names = Values` pattern to map names to values (which we term *definition mode*.

An edge-case arises when defining structure pattern is nested in a selecting names pattern like:
```
x.(a, b.(t = 1), c)
```

The crux of the issue is what does this above expression return?
There are a few reasonable options:
* `None`
* `(a, b, c)`
* `x`

In order to assure syntax composes well Elder chooses the 3rd option.

This forces the syntax to be orthogonal.
One can either be selecting or defining a structure but not both.

Elder enfoces this by poisoning the parents when any names are assigned (using `=`).
This means all parents above whatever name is assigned will not be in *selection mode*.

Here's a deeply nested example:
```
o.(x, y/(a, b:(t, u = 0, v) = 1, c) = 2, z) = 3

// is equivalent to:
o = 3
  .x
  .y = 2
    /a
    /b = 1
      :t
      :u = 0
      :v
    /c
  .z
```

Notes:
* The most deeply nested assignment is `u = 0` which means all parents of `u` will be in *definition mode*
* The parents of `u` are `b, y, o`
* Since the parents of `u` are in *definition mode* we don't select their children. Instead each parent is emitted which allows us to assign to each parent.

Contrast the same example but without any assignment:
```
o.(x, y/(a, b:(t, u, v), c), z)

// is equivalent to (when fully expanded):
(o.x, o.y/a, o.y/b:t, o.y/b:u, o.y/b:v, o.y/c, o.z)
```

Notes:
* Since there's no assignment the entire expression is in *selection mode*
* When in *selection mode* an expression returns it's children but not itself. This is purposely orthogonal to *definition mode*
* The entire *selection expression* can be mapped to series of R-hand values

These can be mixed:
```
(x.(a = 0), y.(c, d, e), z.(f = 1)) = (2, 3, 4, 5, 6)

// is equivalent to:
x = 2
  .a = 0
y
  .c = 3
  .d = 4
  .e = 5
z = 6
  .f = 1
```

Notes:
* This works b/c *definition mode* only affects its parents. In this example this means only `x` and `z` are in *definition mode*
* `y` is a sibling to `x` and `z`
* `y` is in *selection mode* because none of it's children have assignment



## Issues, Edge-Cases, Ambiguities, and Quirks
------------------------------------------------------------------------------------------------------------

### Identifier vs Value Structure

TODO?
* Should we require that nested structures use identifiers? that ways we can avoid values?
  * or rather if it is a value then it is implicitly a name?

### Keys vs Names

```
o.
  3
  b = 4
  5

o.(0, b, 2) = 3, 4, 5
```

```
o.
  a = 1
  'b'
  "c"
  2
  d
    3
    4
    5

o.(a = 1, 'b', "c', 2, d/(3, 4, 5))

o
  .a    = 1
  .None = 'b'
  .None = "c"
  .(2)
  d
    3
    4
    5
```

### Selection


### Relator Chaining

// TODO:
// - Is an effect of orthogonal syntax?
// - 

### 

* Relator edge-cases
  * x.(a) = 1
  * chaining `x.(a = 1) = 0` vs `x.(a = 1)` vs `x.a = 1` vs `x.(a, b:(t, u = 1, v), c) = (0, 1, 2)`
  * nested assign `x.(a, b = 1, c) = 2` vs `x.(a, b.(t = 1), c) = 2` vs `x.(a, b.(t = 1), c) = (2, 3, 4)`?
    * should matching pair w/ the parens?


## QnA
------------------------------------------------------------------------------------------------------------

### Reserved

### Pattern of: Path + Select = Values

### Definition, Mutation, and Scope

### Default (eg \n + INDENT)

### Define Structure vs Equals Sequence

### Comptime vs Runtime Indexing

### Relator for Structured Keywords (eg `:type`)



## Summary
------------------------------------------------------------------------------------------------------------

### Notation

All of the notation used so far:
```
.         // Object-Property Relator
/         // Parent-Child Relator
:         // Data-Metadata Relator
```
