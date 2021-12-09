---
layout: default
title: Base Sugar
description: Sugar for [Tozen Base](./base)
permalink: base-sugar
---

[Tozen Base](./base) is used by programs and not meant for developers to use directly.

To make it more usable we add sugar which will emit [Tozen Base](./base).
This makes the syntax more ergonomic for developers while allowing the [Tozen Base](./base) to stay minimal and consistent.


## Alias for `None` using `_`
------------------------------------------------------------------------------------------------------------

Instead of fully specifying `None` a developer can use a single underscore `_`.
This is a common pattern in many languages which should be familiar to many developers.

`_` is useful b/c it is:
* visually distinct from alpha-numeric characters
* used in many languages to represent concepts like unbounded names
* more terse than `None`

For values, use it in place of a item:
```
items-0 = (1, _, 3)             // means 'None' at middle position
items-1 = (a = 1, b = _, c = 3) // 'b' has a name w/ value 'None'
```

Although often not useful; use it to represent a nameless entry:
```
items-0 = (_ = 1)               // equivalent to 'items-0 = 1'
items-1 = (a = 1, _ = 2, c = 3) // equivalent to 'items-1 = (a = 1, 2, c = 3)'
items-2 = (_ = 1, _ = 2, _ = 3) // equivalent to 'items-2 = (1, 2, 3)'
```


## More Delimiters
------------------------------------------------------------------------------------------------------------

To maximize readability it is useful to have multiple delimiters.

It allows a developer to write less code while focusing on only the data.
At other times it allows for a developer to make visually distinct formatting of their data to make it's shape clear.

The delimiters are (ordered by precedence):
* Space     ` `
* Comma     `,`
* Semicolon `;`
* Pipe      `|`
* Newline   `\n`

Delimiters are ordered by this precedence b/c it followes the natural ordering used in most languages. 
The goal is that the delimiters will match the readers expectations.

We can represent a record in a few ways:
```
rec-space     = (1 2 3)
rec-comma     = (1, 2, 3)
rec-semicolon = (1; 2; 3)
rec-pipe      = (1 | 2 | 3)
rec-newline   =
  1
  2
  3
```

Since there is only one level used per record they all result in the same record.

### Assign Precedence

Assignment precedence is greater than space delimiter but less than comma.
This means it's possible to drop parentheses when assigning a record in some cases.

Here's how it works:
```
rec-0 = 1 2 3      <desugars to>  rec-0 = (1, 2, 3)
rec-1 = 1, 2, 3    <desugars to>  (rec-1 = 1), 2, 3
rec-2 = 1 2, 3     <desugars to>  (rec-2 = 1, 2), 3
rec-3 = 1, 2 3     <desugars to>  (rec-3 = 1), (2, 3)
rec-4 = (1, 2, 3)  <desugars to>  rec-4 = (1, 2, 3)
```

### Multiple Delimiters

When delimiters are mixed it causes them to infer different groupings using `()`.

Every delimiter will desugar to only `,` and `()`.

A few examples makes the rules clear:
```
rec-0 = (1 2, 3, 4 5 6)  <desugars to>  rec-0 = ((1, 2), 3, (4, 5, 6))
rec-1 = (1 2, 3; 4 | 5)  <desugars to>  rec-1 = ((((1, 2), 3), 4), 5)
rec-2 = (1 | 2; 3, 4 5)  <desugars to>  rec-2 = (1, (2, (3, (4, 5))))
```

### Example - Table

It's rare to mix many delimiters except in cases where you're building complex structures.

Consider describing a staffing table:
```
staff =
  'name'      | 'title'                | 'age' | 'scheduled courses'
  'Billy Bob' | 'Teacher Assistant'    | 27    | 'Building B', 'Room 16A'; 'Building A', 'Room 8C'
  'Jill Jack' | 'Professor of History' | 38    | 'Building A', 'Room 3C'
  'Mary May'  | 'Director'             | 62    | 'Building A', 'Room 46E'; 'Building D', 'Room 9F'
```

using the precedence rules this desugars to (whitespace added for clarity):
```
staff =
  'name',      'title',                'age', 'scheduled courses'
  'Billy Bob', 'Teacher Assistant',    27,    (('Building B', 'Room 16A'), ('Building A', 'Room 8C'))
  'Jill Jack', 'Professor of History', 38,    ('Building A', 'Room 3C')
  'Mary May',  'Director',             62,    (('Building A', 'Room 46E'), ('Building D', 'Room 9F'))
```

Notice that:
* The above version is more clear as everything clearly aligns and less nested parentheses to visually parse.
* Parentheses can always be used so if you don't want to use all the delimiters there's no need to do so.
  They're optional and should only be used to make code more clear.
* This can be made simpler if we had a way to infer identifiers as some type of String.
  This will be introduced later to simplify further.


## Data Inference Operator using `:`
------------------------------------------------------------------------------------------------------------

In many languages `:` is used to describe mapping a key to a value like:
```
defn:
  age:      27
  name:     Bob T. Biggs
  location: 206 Bridgeton Ave
```

This is ergonomic for developers as they don't have to specify everything and makes the syntax more succicient.

Tozen mimics this syntax with a few modifications:
* Instead of attaching `:` to a name it's an operator similar to `=`
* `:` desugars to `=` after inferring it's type
* Terminates on `,` or any lower precedent delimiter (`; | \n`)
  * This implies `:` is usable for defining multiple keys-values inline just like `=`

The primary usefulness of `:` operator is how it interprets lexemes into literals.
The process is very conservative in order to make it predictable and consistent.

Here are the steps:
* Group the R-hand until reach `,` or lower precedent delimiter (`; | \n`)
  ```
  x-0 : 1, 2, 3  <groups-by>  (x-0 : 1), 2, 3
  x-1 : 1, 2 3   <groups-by>  (x-1 : 1), (2, 3)
  x-2 : 1 2, 3   <groups-by>  (x-2 : (1 2)), 3
  x-3 : 1 2 3    <groups-by>  (x-3 : (1 2 3))
  ```
* Infer the type from the entire grouping of the R-hand after `:`.
* If the type is a known primitive value is left alone:
  ```
  x : 4       =>  x = 4      // Integral
  x : 1.2     =>  x = 1.2    // Decimal
  x : 3./4    =>  x = 3./4   // Fraction
  x : 98.75%  =>  x = 98.75% // Percent
  x : 0b1010  =>  x = 0b1010 // Binary
  x : 0o744   =>  x = 0o744  // Octal
  x : 0xF00D  =>  x = 0xF00D // Hex
  x : 4e-2.5  =>  x = 4e-2.5 // Exponent
  x : True    =>  x = True   // Boolean True
  x : False   =>  x = False  // Boolean False
  x : None    =>  x = None   // None Literal
  ```
* String-like literals are left alone:
  ```
  verbatim-str : 'a b c'  =>  verbatim-str = 'a b c'
  literal-str  : "a b c"  =>  literal-str  = "a b c"
  ```
* Identifiers are converted into String literal and joined
  ```
  x : a, b, c  =>  x = "a", b, c
  x : a, b c   =>  x = "a", (b, c)
  x : a b, c   =>  x = "a b", c
  x : a b c    =>  x = "a b c"
  ```
* Parentheses distribute just like `=`
  ```
  // `()` allowed
  x : 1, 2, 3   => (x = 1), 2, 3
  x : (1, 2, 3) => x = (1, 2, 3)

  // `()` cause distribute to each value
  x : a, b, c   => x = "a", b, c
  x : (a, b, c) => x = ("a", "b", "c")

  // `()` cause distribute to each value and group spaces
  x : a b, c d e, f g   => x = "a b", c d e, f g
  x : (a b, c d e, f g) => x = ("a b", "c d e", "f g")

  // `()` distribute even to nested records
  x : a b, (c, d e, f), g h   => x = a b, (c, d e, f), g h
  x : (a b, (c, d e, f), g h) => x = ("a b", ("c", "d e", "f"), "g h")

  // Even complex precedence distribute over `()`
  x : a b, c; d e, f | g   => (((x = "a b"), c), ((d, e), f), g)
  x : (a b, c; d e, f | g) => x = ((("a b"), "c"), (("d e"), "f"), "g")

  // `:` applies to all of it's values when inline
  // - This helps `:` easy to use b/c it must be applied to all of it's values
  x : 1, b = t u, c = 2   => (x = 1), b = t u, c = 2
  x : (1, b = t u, c = 2) => x = (1, b = "t u", c = 2)

  // `:` can be nested and it applies to only it's values
  x = 1, b : t u, c = 2   => x = 1, b = "t u", c = 2
  x = (1, b : t u, c = 2) => x = (1, b = "t u", c = 2)
  ```
* Everything too complex is considered a String literal
  ```
  // A mix of integers and identifiers so too complex
  x : 1 + 2  =>  x-0 = "1 + 2"

  // Mix of identifiers and String-like is too complex and String-like are escaped
  x : a b 'c'  =>  x = "a b \'c\'"
  x : a b "c"  =>  x = "a b \"c\""

  // Mix of digits and identifiers is too complex
  x : (3 feet, 2 inches)  =>  x = ("3 feet", "2 inches")

  // Height identifier is unclear so is escaped as in StringLiteral
  x : (Briane Biggs, 36, 5'8'')  =>  x = ("Briane Biggs", 36, "5\'8\'\'")
  ```
* It only works inline
  ```
  // Notice b => "b"
  x : (1, b, 2)  =>  x = (1, "b", 2)

  <vs>

  // but isn't changed here b/c isn't inline
  x :  =>  x =
    1        1
    b        b
    2        2
  ```
* It applies to it's entire R-hand until terminal is reached
  ```
  x : (1, a = (2, u, v = (f, g), 3), 4)  =>  x = (1, a = (2, "u", v = ("f", "g"), 3), 4)
  ```
  Notice although `u`, `f`, `g` values are deeply nested they are inferred.

The inference process is conservative to make it more predictable and consistent.

If a developer wants something more complex then they will need to write the fully qualified version.

### Edge Cases

TODO


## Anonymous Children
------------------------------------------------------------------------------------------------------------

There are cases where we want to represent a record without a meaningful name.

For example, consider the JavaScript code:
```
let myElems = [
  [ a, [ b, c ], d ],
  [ i, [ j, k ], l ],
  [ w, [ x, y ], z ]
];
```

With our existing syntax we can model this in a similar way:
```
my-elems =
  a, ( b, c ), d
  i, ( j, k ), l
  w, ( x, y ), z
```

However we can do better as this is starting to get noisy and harder to visually parse.

To do so, we'll introduce a new syntax which allows us to define an anonymous child.
Here are the rules:
* The syntax is either `-` or `*` or the child relator `/`
  * At this point it's the developer's choice as to which syntax to use (although they must do so consistently).
* The same syntax must be used consistently for children at the same level
* The syntax must be at the start of a line (ie after whitespace)
* The anonymous item indicator is syntax only (just like parentheses) to inform the developer and compiler

Using this we can expand our previous syntax to better model the structure of the data:
```
my-elems =
  - a, 
    b, c
    d
  - i
    j, k
    l
  - w
    x, y
    z
```

and even more expanding:
```
my-elems =
  - a, 
    * b
      c
    d
  - i
    * j
      k
    l
  - w
    * x
      y
    z
```

Notice that we keep the same syntax for children at the same level.
This is enfoced b/c it's much more clear and easier to visually track.

### Layout

To assure the grouping is clear and terse, we require the children of the anonymous item to immediately follow it and align to that position like:
```
my-elems =
  - a
    b
    c
  - x
    y
    z
```

and not:
```
my-elems =
  -
    a
    b
    c
  -
    x
    y
    z
```

### Nesting

Items can be nested arbitrarily as long as they're consistent.

Here's one example:
```
my-elems =
  - - a
      b
    - c
  - - i
      j
    - k
  - - w
      x
    - z
```

This is read as:
* a sequence of 3 elements
* each contains a sequence of 2 elements
* the first contains a sequence of 2 elements
* the second constains a sequence of 1 element

This can be rewritten to other forms as well which work well for non-deeply nested structures:
```
my-elems =
  - a,b
    c
  - i, j
    k
  - w, x
    z
```


## Record
------------------------------------------------------------------------------------------------------------

### Multiline Assign

Sometimes it makes sense to not have the value on the same line as the `=`.

Essentially the child or children become the value of the R-hand of the expression.

There's a few cases where the values is:
* A single value
  ```
  x =
    1
  ```
  * This is equivalent to `x = 1` since `1` is the only immediate child of `x`
  * Often useful when the value is a self-executing expression
* A single, complex value
  ```
  x =
    a =
      1
      2
  ```
  * This associates `x` name to the structure starting with `a` since `a` is the only immediate child of `x`
  * Often useful when the value is a complex data-structure
* Multiple values
  ```
  x =
    1
    2 
    3
  ```
  and equivalently:
  ```
  x
    =
      1
      2
      3
  ```
  * This is equivalent to `x = (1, 2, 3)`
  * Often useful when you want to store a literal, multi-valued, container like `List`, `Sequence`, `Map`
  * Could also be helpful if you're defining the value as well as other relators

### Multiple Values

Due to precedence, parentheses must be used for multiple values:
```
x         = (1, 2, 3)             // Assign name 'x' to simple record
x         = (a = 1, b = 2, c = 3) // Assign name 'x' to more complex record
(x, y, z) = (1, 2, 3)             // Assign each name to each value like: (x = 1), (y = 2), (z = 3)
(x, y, z) = (a = 1, b = 2, c = 3) // Assign each name to each record entry like: x = (a = 1), y = (b = 2), z = (c = 3)
```

It is illegal to set multiple names to a single value.
For consistency, they must always be equal in arity:
```
(x, y, z) = 1              // Illegal
(x, y, z) = (1, 2)         // Illegal
(x, y, z) = (1, 2, 3)      // Works
(x, y, z) = (1, 2, 3, 4)   // Illegal
(x, y, z) = (1, 2, (3, 4)) // Works b/c now element arity matches which causes: z = (3, 4)
```


## Relators
------------------------------------------------------------------------------------------------------------

### Inline Definition Syntax

Inline data definition can quickly become unreadable.
It requires the developer to visually track nested expressions.

To avoid confusion, only a constrained syntax form is allowed inline.
It is purposely constrained to have minimal visual noise and avoid complexity.

Here are a few examples:
```
// Name with single relator
x .(a = 1)

// Name with value and single relator
x .(a = 1) = 0

// Name with value and multiple relators
x .(a = 1) :(b = 2) /(c = 3) = 0

// Path to name with value and multiple relators
x.y:z .(a = 1, 2, b = 3) :(c = 4, 5) /(6, 7) = 0
```
Notice that:
* the name is always at the start of the line (the leading name)
* the leading name is what is being defined
* each relator is delimited by spaces
* each relator uses parentheses
* the contents of each relator is a record
* the value for the leading name is always last

In order to keep the syntax clear we don't allow nesting on the same line.
Syntax like this is invalid:
```
// 'a' cannot be nested in 'x' within inline form
x .(1, a :(2, 3, 4), 5) = 0
```

and would need to be converted to the multiline form like:
```
x = 0
  .
    1
    a :(2, 3, 4)
    5
```

An effect of this is that this sugar only works for the start of a line.
This keeps the syntax more clear and consistent.
Which means developers spend less time visually tracking nested expressions.

### Shape of Data

Code is easier to understand when the shape of the code follows the shape of the data. 
Especially with minimal syntactical noise.
Relators make this easier to do as their syntactical patterns match the model.

With a few examples the rules become obvious.
All of the following examples in different forms are equivalent.

Form useful when when want to visualize structure:
```
x
  :
    y
      .
        a = 0
        b = 1
        c = 2
```

Form useful when mixing multiple relators together:
```
x
  :y
    .a = 0
    .b = 1
    .c = 2
```

Form useful when path to elements less important:
```
x:y.
  a = 0
  b = 1
  c = 2
```

Form useful to mix paths with structure:
```
x:
  y.
    a = 0
    b = 1
    c = 2
```

Form useful inline and grouped together to better visualize common relators:
```
x:y .(a = 0, b = 1, c = 2)
```

Flexibility is allowed because there are many different shapes to data.
We should allow the developer to model the structure in the most direct and applicable way.

In general, developers should prefer the structure which best models the data layout as directly as possible.

### Default Relator

When no relator is provided, the default is the parent-child relator `/`.
Indented elements indicate a parent-children relationship.

Many languages use indentation or `/` to indicate children.
Tozen uses the same convention.

These are equivalent:
```
// Explicit `/`
x
  /y
    /z
      /a = 0
      /b = 1
      /c = 2

// Default to use implicit `/`
x
  y
    z
      a = 0
      b = 1
      c = 2
```

It's impossible to represent default relators inline.
You must use `/`:
```
x/y/z /(a = 0, b = 1, c = 2)
```

In later layers it is possible to change the default relator.

### Path

One aspect that's unique to relators is that they're used to create paths in order to:
* define data
* abstacts over concepts like file and network paths
* traverse data structure
* operate somewhat like queries

Consider a somewhat complex example which uses both the `.` (object-property) and `:` (data-metadata) relators:
```
o
  .a = 0
  .b
    :x = 1
    :y = 2
  .c = 3
    :z = 4
```

Using paths we can access and define this data in multiple ways.
Syntactically paths act more like other languages and uses the relator as a delimiter.

For example, to select `x` in `o` we write:
```
o.b:x
```
By default, selecting a value will return it's result.

It's also useful in when paired with equals syntax:
```
o.b:x = 3
```
This is read the same as the previous example however the final step will assign `3` to `x`.

Since this level doesn't support variables, it's not allowed to use select for the R-value.

For example, this isn't allowed:
```
// L-value is path to 'x'
// R-value is path to 'z'
o.b:x = o.c:z
```
At higher levels variables are allowed.

### Terse Data Modeling

Paths also allow for terse data modeling.

Consider a realistic example to set deeply nested values:
```
html
  body
    header
      .
        style
        .
          css
            .
              width  = 100%
              height = 20px
```

There's extra structure and visual noise which are necessary for HTML.
However they distract from the real task of setting `width` and `height`.

Using paths we can do better and remove most of the syntactical noise:
```
html/body/header.style.css.
  width  = 100%
  height = 20px
```

We can use paths to collapse the heirarchy from above to what we really are concerned about.


## Examples
------------------------------------------------------------------------------------------------------------

### Serialization Languages (like JSON, YAML, SDLang)

Using the syntax so far we have enough to emulate various serialization languages.

JSON is ubiquitous so let's see the difference.

Consider a somewhat simple JSON example:
```
[
  {
    "department": "History",
    "staff": [
      {
        "name": "Jane Janison",
        "is-tenured": true
      },
      {
        "name": "Billy Bane",
      },
      {
        "name": "Katie Kattson",
        "is-assistant": true
      },
    ]
  },
  {
    "department": "Mathematics",
    "staff": [],
  },
  {
    "department": "Biology",
    "staff": [
      {
        "name": "Dirk Darwin",
        "specialization": "Evolutionary",
        "teacher-assistants": [
        ]
      },
      {
        "name": "Ellen Erickson",
        "teacher-assistants": [
          {
            "name": "John Johnson"
          }
        ]
      },
    ],
  },
]
```

We can recreate with Tozen:
```
[
  - department : History
    staff = [
      - name         : Jane Janison
        is-tenured   : True
      - name         : Billy Bane
      - name         : Katie Kattson
        is-assistant : True
  - department : Mathematics
    staff      : []
  - department : Biology
    staff = [
      - name               : Dirk Darwin
        specialization     : Evolutionary
        teacher-assistants : []
      - name : Ellen Erickson
        teacher-assistants = [
          name : John Johnson
```

Although it is more terse, it's not perfect.
There's more to do in order to remove repetition in the shape and structure of the data.

Later on it can be simplified further as we introduce new concepts.

## HTML 

We can model some simple declarative languages as well.

Consider a simple header in HTML:
```
<html>
  <body>
    <div class="header home-header" style="width = 100%, height=20px">
      <h2>Home</h2>
      <p class="header-description home-header-description" style="font-size=18">Welcome Home</p>
    </div>
  </body>
</html>
```

We can rewrite in the multiline form:
```
html/head/div
  .class : header home-header
  .style
    .width  : 100%
    .height : 20px
  h2 : Home
  p  : Welcome Home
    .class : header-description home-header-description
    .style
      font-size = 18
```

as well as a more compact inline form:
```
html/head/div.(class : header home-header, style.(width : 100%, height : 20px))
  h2 : Home
  p  : Welcome Home
    .class : header-description home-header-description
    .style
      font-size = 18
```

Like the serialization example there's still some visual noise but it's still relatively terse.
In later sections we'll add more abstractions to reduce the noise.


## Summary
------------------------------------------------------------------------------------------------------------

This document describes the sugar for [Tozen Base](./base).

It is flexible enough to express data and serialization languages like JSON, YAML, SDLang, etc.
It can also expressive to handle some declarative languages like HTML.

The next document will introduce the next layer of Tozen which guarantees it will be fully erased by the compiler.

### Notation

New notation:
```
_         // Alias for `None`
` `       // Space Delimiter
;         // Semicolon Delimiter
|         // Pipe Delimiter
\n        // Newline Delimiter
:         // Data Inference Infix Operator
-, *      // At start-of-line indicate anonymous item
```

### Precedence

The updated precedence table (high to low):
* Parentheses
  * eg `x = (1, 2, 3)`
* Inline Relator
  * eg `o.p:q/r`
* Prefixed, Unary Operator
  * eg `-4`, `+4`
* Space Delimiter
  * eg `x = 1 2 3`
* Infix Assign & Infer
  * eg `x = 1 2 3` `x : 1 2 3`
* Comma Delimiter
  * eg `x = 1, 2, 3`
* Semicolon Delimiter
  * eg `x = 1; 2; 3`
* Pipe Delimiter
  * eg `x = 1 | 2 | 3`
* Newline Delimiter
  * eg
    ```
    x =
      1
      2
      3
    ```
