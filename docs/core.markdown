---
layout: default
title: Core IR
description: Core layer of Tozen for data representation
permalink: core
---

This document describes the lowest level of the Tozen language which is primarily useful for data modeling, data representation, and serialization.
It's expressiveness roughly correlates with languages like JSON.

Many higher level syntaxes are more expressive but ultimately reduce to this core syntax.
Keeping the set of core elements simple makes it easier for tools to be developed for it.
It is also rare that a developer will use it directly as the higher level syntaxes are more ergonomic.


## Literals
------------------------------------------------------------------------------------------------------------

Tozen has a few literals types and values which have a universal interpretation.

This doesn't mean these values can't be cast (or interpreted) to other types but that the default interpretation must be consistent.

### Literal Values

These are values which are interpreted as known types to avoid unnecessary visual noise:
* `Numeric` Literals
  * Integral `4`
  * Decimal  `1.2`
    * Floating point
  * Fraction `22./7`
    * Exact representation
  * Percent  `98.75%`
    * Fixed point
  * Binary   `0b1010`
    * `b` must be lowercase to be more visually distinct from digits
  * Octal    `0o744`
    * `o` must be lowercase to be more visually distinct from digits
  * Hex      `0xF00D`
    * Hexidecimal alpha characters must be uppercase to make them visually distinct
    * `x` must be lowercase to be more visually distinct from digits and hex
  * Exponent `4e-2.5`
    * `e` must be lowercase to be more visually distinct from digits
    * exponent can be Integral, Decimal, or Fraction
* `Bool` Literal
  * Only possible values are `True` and `False` literal values
* `None` Literal
  * Represents no value
  * Both value and type are `None`
  * Not castable to other types (like `NULL` in most languages)

### No Value

To represent no value we use `None`.

`None` is used to conceptually differentiate from the common concepts of `Null` and `Nil` used by other languages.

`None` is different in that:
* It's both a global and immutable
* Both its value and type are `None`
* It is not castable to other types.
  Since `None` isn't castable it must be manually handled instead of being implicitly cast (often using a `Or` types and its variants).

### Quoting

There are two types of quoting:
* `Verbatim`
  * uses single quote character `'`
  * interprets the characters exactly as they are in source (ie no escaping)
  * useful for representing foreign text or text that will be processed as text (not as code) like:
    * embed inline Assembly, HTML, unstructured quote, etc.
    * embed binary, Base64, etc.
    * embed String template and post-process while treating as text (much like C-preprocessor)
  * UTF8 characters are copied directly
* `StringLiteral`
  * uses double quote character `"`
  * allows for escape characters (eg `"Hello \t World"`)
  * can escape quotes as well (eg `"Hello 'Wide' \"World\""`)
  * useful for representing Strings in the language
  * UTF8 characters are copied directly

Both follow common patterns (for simplicity we'll use StringLiteral `"` for the common examples):
* When the opening quote is repeated 1 or 2 times it will implicitly close at the end of the line (or on closing quote)
  * all of these produce the same output
    ```
    // Common inline String literal
    msg-0 = "Hello World"

    // Implicitly close at EOL
    msg-1 = "Hello World

    // Common inline String literal
    msg-2 = ""Hello World""

    // Implicitly close at EOL
    msg-3 = ""Hello World
    ```
* When the opening quote is repeated 3 or more times it can be used for inline or block depending on it's content
  * inline String literal
    ```
    // Common inline String literal
    msg-0 = """Hello World!"""

    // Escapes fewer number of quotes
    msg-1 = """Hello "World"!"""

    // Leading and trailing spaces are preserved (like usual)
    msg-2 = """  Hello "World"!  """

    // When inline mode, implicitly close at EOL
    msg-3 = """Hello "World"!
    ```
  * Block String literal
    ```
    // If whitespace after, assume block String literal after indent on next line
    msg-0 = """
      Hello World!

    // Escape characters
    msg-1 = """
      Hello \n """World"""!

    // Use `|` to indicate significant leading and trailing whitespace
    // - If leading whitespace, `|` must be used for every line including those w/o leading whitespace
    // - If trailing whitespace, `|` only necessary for lines which have significant whitespace
    // - Only meaningful when in Block mode and at start and end of the content of each line
    // - Translates to ""  Hello "World"  \nof               \n     tommorrow!""
    msg-2 = """
      |  Hello "World"  | // Leading and trailing whitespace
      |of               | // Only trailing whitespace
      |     tommorrow!    // Only leading whitespace

    // Escape at EOL indicates continue line and replaces with single space
    // - Translates to "Hello World of tommorrow!"
    msg-3 = """
      Hello World    \
      of             \
      tommorrow!

    // Can mix significant whitespace with line continue to preserve whitespace
    // - Translates to ""  Hello "World"   of                     tommorrow!""
    msg-4 = """
      |  Hello "World"  | \
      |of               | \
      |     tommorrow!
    ```
* When inline, the greater number of quotes escape fewer number
  ```
  // Opening `""` escapes inner `"`
  msg-0 = ""Hello "World"!""

  // Opening `""""` escapes inner `"`, `""`, and `"""`
  msg-1 = """"Hello "Wide" ""Wide"" """World"""!""""
  ```
* Block quoting only ends at dedent
  ```
  // Once in Block mode, all quotes are escaped until dedent
  msg-0 = """
    "Hello" ""Wide"" """Wide""" """"World""""!

  // All elements in Block assume String mode (Verbatim or StringLiteral)
  msg-1 = '''
    "Hello \n World" // This will not escape b/c is Verbatim Block

  msg-2 = """
    'Hello \n World' // This will escape b/c is StringLiteral Block
  ```


## Record
------------------------------------------------------------------------------------------------------------

The core syntactical structure is a record.

Records are similar to C-like structs but more flexible.
Records represent a syntactical pattern which is used in many contexts. 
Each context interprets the record in a different way (much like S-Expressions).

Records are:
* an ordered sequence of elements
* siblings are elements at the same scope
* siblings are indexed and start at index 0
* values are optionally named
* values can be accessed by name or index
* heterogenous
* often immutable (but not always b/c it depends on the contexts the syntax is used in)
* finite in number
* purely syntax
* name identifiers are interpreted as literals
* the same name can be repeated (depending on the context)

The simplest is the multiline form:
```
x = "a" // name 'x' with value "a" at index 0
'y'     // no name with value 'y' at index 1
z = 3   // name 'z' with value '3' at index 2
```

This can be made inline by using `,` instead of `\n`:
```
x = "a", 'y', z = 3
```

This works b/c `,` is lower precedence than `=`.

### Illegal to mix inline and multiline siblings

In order to make code consistent and more readable there are rules enforced:
* It is illegal to nest a multiline form in a inline form
* It is illegal to mix inline and multiline for siblings

These rules avoid many cases of ackward and irregular layout which would otherwise need to be visually tracked.

Consider the following record:
```
x = 0
y =
  a = 1
  b = 2
  c = 3
z = 4
```

These are invalid representations of the record:
```
// Invalid b/c multiline form can't be nested in an inline form
x = 0, y =
  a = 1
  b = 2
  c = 3
z = 4

// Invalid b/c `a, b, c` are siblings but mixed form
x = 0, y = (a = 1,
b = 2, c = 3), z = 4

// Invalid b/c `x, y, z` and `a, b, c` are siblings but mixed form
x = 0
y = (a = 1, b = 2,
  c = 3), z = 4
```

while these are valid:
```
// Everything inline works
x = 0, y = (a = 1, b = 2, c = 3), z = 4

// Allowed b/c `a, b, c` are all siblings and children of `y`
// Thus they are not siblings of `x, y, z`
x = 0
y = (a = 1, b = 2, c = 3)
z = 4
```

### Valid Names

Names must be formatted in a few ways:
* Are not quoted like `x = 2`
* Must start with a ASCII alpha lowercase character (`a - z`)
  * After start can contain `a - z` or `0 - 9` or dashes `-`
* Instead of camel-case (`myLongName`) or snake-case (`my_long_name`) dashes are used (`my-long-name`)
  * Dashes can't start (`-my-long-name`) or end (`my-long-name-`) an identifier
* Nameless records use `None` name
  * eg `x = (None = 2)` is equivalent to `x = 2`
* Similarly, records without values use `None`
  * eg `x = (a = None)`
* `Verbatim String` only when there are spaces or reserved word (see below)
* All records are implicitly numbered by their index starting at `0`

### Verbatim Name

We allow for spaces in record names but only by specifying they should belong to a single unit.

To indicate that a name should have a space wrap it a `Verbatim String` (using `'`).
We don't use a `StringLiteral` (uses `"`) b/c it allows escaping which we don't want for a static name.

When `Verbatim String` is used there must be a space in the name or it must be reserved word.
If there isn't a reason to use a `Verbatim String` for a name then it shouldn't be used.
A simple identifier should is preferred.

Here's a simple example:
```
staff = 
  'Billy Bob'   = (position = 'dev',   age = 28, weight-lb = 183, height-inch = 68)
  'Janey Jane'  = (position = 'IT',    age = 36, weight-lb = 132, height-inch = 72)
  'Will Ozmand' = (position = 'admin', age = 18, weight-lb = 115, height-inch = 60)
```

Here we create a data description of a sample of staff.
Each staff name point to records which hold their properties.

### Nesting

Records can be nested using indentation.

Only two spaces per indent are allowed.
Two spaces are used b/c it's the minimal amount to make it clear there's indentation.
No tabs, no variable spaces, nothing but two spaces per indentation.

```
x =
  0
  y =
    1
    z = 
      2
      3
      4
    5
  6

// equivalent to:
x = (0, y = (1, z = (2, 3, 4), 5), 6)
```

Since `,` is higher precedence than `\n` we're able to represent simple graphs:
```
// `a, b` are both parents of children `1, 2, 3`
a, b =
  1
  2
  3
```

### Missing Values

It is illegal to not provide a value.

Each of these fail:
```
a =     // Fails b/c no value present
b = ,   // Fails b/c only delimiter
c = 1,  // Fails b/c trailing delimiter w/o value
d = , 1 // Fails b/c leading delimiter w/o value
```


## Relator
------------------------------------------------------------------------------------------------------------

It is useful to model multiple aspects of the same concept.

Consider modeling a car by its:
* ***attributes***: `weight`, `color`
* ***metadata***: `make`, `model`, `year`
* ***parts***: `wheel`, `engine`, `door`

The aspects of the car are its ***attributes***, ***metadata***, and ***parts***.
Examples of its fields of each aspect are `weight`, `make`, and `wheel` respectively.

Modeling in this way is common so Tozen formalizes a mechanism to do so succiciently.
This formalizism captures the *relationship* between a *object* and *descriptors* of that *object*.
Since we are capturing a type of relationship, we coin a new term `Relator`.
A new term also helps conceptually differentiate it from other tools in other languages as it functions slightly differently.

`Relators` work much like a `namespace` comprised of punctuation.
Each relator has a semantic meaning associated to it which is the relationship it represents.
Within a relator, a record is used to describe the fields of that relationship.
Functionally, a `Relator` is a `Record` that lives under specific punctuation with a few syntactical rules.

At this level, there are a small number of `Relators`:
* `.` the dot relator for object-property relation like:
  * for OOP (and many other) languages, accessing the fields of a instance
  * for HTML, accessing a attribute of a DOM element
* `:` the meta relator for data-metadata relation like:
  * used to represent metadata of any data
  * does the work of describing concepts like types, constraints, traits, interfaces, rules, etc.
* `/` the child relator for parent-child relation like:
  * for HTML, a DOM child element
  * for namespaces, a child namespace
  * for filesystem, a directory to it's contents

Continuing the car example, we use relators to match the shape of the data:
```
car
  // Indicates object-property relation
  .
    weight = 1024
    color  = 'Red'

  // Indicates data-metadata relation
  :
    make  = 'Honda'
    model = 'Accord'
    year  = 2012

  // Indicates parent-child relation
  /
    'wheel'
    'engine'
    'door'
```

### Syntax Forms

Without sugar, relators only have a single multiline form.
```
o = 0
  .
    a = 1
    b = 2
    c = 3
  :
    d = 4
    e = 5
  /
    f = 6
    g = 7
    h = 8
```

Notice that:
* Relators are used directly to indicate the shape and structure of the data.
* The body of a relator (after the punctutation) is a Record.
* A relator doesn't use `=` and instead is placed directly after or child of a name.
  This is because records are not values; they are more like namespaces.
  They describe a specific aspect of a name.

### Record-Relator Corresondance

Records and relators can be described as one another.

This is possible because:
* Records are a list-like structure.
* List-like structures have children.
* Relators can describe a parent-child relation using `/`.

Consider a record in both its forms:
```
// Inline
r = (a = 1, b = (c = 2, 3, 4, d = (e = 5), f = 6), g = 7)

// Multiline
r =
  a = 1
  b =
    c = 2
    3
    4
    d =
      e = 5
    f = 6
  g = 7
```

By using the parent-child relator this translates to:
```
r
  /
    a = 1
    b
      /
        c = 2
        3
        4
        d
          /
            e = 5
        f = 6
    g = 7
```

Notice that:
* `/` is used whenever a record has:
  * multiple children like `a = (1, 2, 3)`
  * a nested record like `a = (b = 1)`
* `/` is not used whenever a name refers to a singular element (similar to a LISP atom)

### Relator and Value Merge

XML, SDLang, KDL, and similar languages have a common quirk.
A developer can model a parent-child relationship in multiple ways.
As children of the node itself.
Or children of the node value when it is a container-like structure.
Both the node and the node value have the same expressive power.
This flexibility has downsides as well like:
* developers must decide in which cases to model children as values or under node (usual method)
* any tool which reads the code must be able to handle both cases
* there are multiple subtrees to parse and reason about (even when most common case is under node)
* any query tools must query the value and children differently b/c is ambiguous

Tozen takes a different approach using a simple rule: the relators of a name and the value merge.
This means it's not possible for both a value and a name's children to refer to different records.

Although one shouldn't write code like this, consider an example:
```
// Value is a Record of `a, b, c` with child relator for `d, e`
my-rec = (a = 1, b = 2, c = 3)
  /d = 4
  /e = 5
```

Since both the value and relators merge this results to any of the following:
```
// as a Record
my-rec = (a = 1, b = 2, c = 3, d = 4, e = 5)

my-rec =
  a = 1
  b = 2
  c = 3
  d = 4
  e = 5

// as a Relator
my-rec
  /
    a = 1
    b = 2
    c = 3
    d = 4
    e = 5
```

The value and the relator both refer to the same record.

If a developer wants then to not merge they simply have to change their model so they belong to different names.
Continuing the same example we can keep them distinct by:
```
// as a Record
my-rec =
  value    = (a = 1, b = 2, c = 3)
  children = (d = 4, e = 5)

// as a Relator
my-rec
  /
    value
      /
        a = 1
        b = 2
        c = 3
    children
      /
        d = 4
        e = 5
```


## Line Continuation
------------------------------------------------------------------------------------------------------------

There are a few options for line continuation:
* `\` continue a single line at the end of a line
* `\\` escapes the backslash
* `\\\` continue every line in a block

Here are some examples:
```
// These are equivalent
rec-0 =
  a, b, c \
  d, e, f \
  g, h, i

rec-1 = \\\
  a, b, c
  d, e, f
  g, h, i

// These are equivalent
str-0 = "a \\ b"

str-1 = 'a \ b'

str-2 = """
  a \\ b

str-3 = '''
  a \ b

// String line continuation strip whitespace and must use `|` for whitespace management.
// These are equivalent.
whspace-str-0 = """
  a \\ b

whspace-str-1 = """
  |a \\ \       // Whitespace is stripped
  | b           // Leading whitespace

whspace-str-2 = """
  |a \\ | \     // Trailing whitespace before `|` and then stripped by `\`
  |b            // Notice no space here

whspace-str-3 = """
  |a     \      // Whitespace is stripped
  | \\ | \      // Trailing whitespace before `|` and then stripped by `\`
  |b            // Notice no space here

whspace-str-4 = \\\ """
  |a            // Whitespace is stripped
  | \\ |        // Trailing whitespace before `|` and then stripped by `\\\`
  |b            // Notice no space here

whspace-str-5 = \\\ """
  |a |          // Trailing whitespace indicated by `|`
  |\\|          // Whitespace stripped by `\\\`
  | b           // Leading whitespace indicated by `|`
```

Generally line continuations should be used for values and not in the structure of a record.
Although all of these equivalent `rec-2` is less readable:
```
rec-0 =
  a = 1
  b = 2
  c = 3
  d = 4
  e = 5

rec-1 = (a = 1, b = 2, c = 3, d = 4, e = 5)

rec-2 = (a = 1, b = 2, \
  c = 3,               \
  d = 4, e = 5)
```


## Comments and Document String
------------------------------------------------------------------------------------------------------------

Comments are for providing freeform information to developers.
They are not meant to be parsed.

Document strings (docstring) are used for providing semi-structured information to developers and other tools.
They are meant to be parsed.

There is only one syntax for comments `//` and docstrings `//-`.
Notice that docstrings are just like comments but w/ a hyphen suffix.
This is useful b/c it aids in versioning and labeling like `//-2.4.5-Project-Title` with clear delimiters.

Comments and docstrings function the same way (for demonstration, I'll only use comments):
* Comments a line
  ```
  // My header
  div .class = header main
  ```
* Inline comment
  ```
  div
    .class = header //( My header ) main
  ```
  Notice how parentheses follow the pattern of start/stop throughout the syntax including comments
* Comments until the end of a line
  ```
  div 
    .class       = header main // My main header
    .style.width = 100%        // Full width
  ```
* Series of comments
  ```
  // Distinct comment 0
  // Distinct comment 1
  // Distinct comment 2
  ```
* Block comments
  ```
  // My Notes
    Notes both ignore the end of a line and it's children.
    So these are part of the comment as well.
  ```


## Invalid Syntax
------------------------------------------------------------------------------------------------------------

Like any language, it's possible to construct invalid and confusing syntax.

Tozen attempts to reduce the likelihood in a few cases.

### Assignment Precedence

It is illegal to nest assignments without clear precedence.
```
x = 1       // Valid
x = a = 1   // Invalid b/c due to precedence reads as `(x = a) = 1`
x = (a = 1) // Valid b/c precedence is explicit again
```

### `None` is `None`

It's invalid to use `None` for both name and value.

Each of these are invalid:
```
None = None

o.
  None = None

o.(None = None)
```

### Integral Literal Record Names

In higher levels of Tozen the indexes of records will be used to query and define.

Due to this lower levels of syntax can't conflict.

At this level, this means `Integral Literals` can't be used for names:
```
o =
  5 = 1
  9 = 2
  6 = 2
```

On top of looking ackward, this will conflict with higher levels of the syntax.

### Mixing `None` with Relators

It is invalid to mix `None` with Relators.

`None` is unique in that it doesn't allow for anything else related to the name.

This is invalid:
```
x = None
  .a = 1
  :b = 2
  /c = 3
```

### Inline Overwrite

Consider the simple example:
```
(x = 0) = 1
```

This is read as:
* Assign `x` to `0`
* Get result of `x = 0` expression
* Assign result to `1`

Elder doesn't allow this syntax and will emit a syntax error.

The reasoning is that code like this will lead to more bugs than useful use-cases.

Instead to assign multiple values to the same name you need to do so using multiple statements:
```
x = 0
x = 1
```

This makes the intent more clear.

It has the added benefit of a user doesn't have to visually track which names are used multiple times as they can be confident they're not redefined.
Even in complex statements this makes it slighly easier to mentally parse.


## Edge-Cases, Ambiguities, and Quirks
------------------------------------------------------------------------------------------------------------

Instead of an unfortunate developer discovering a quirk of the syntax in the middle of their work; we've done our best to discover and make them explicit.

As the syntax grows more complex there will be more quirks which will be make explicit at the end of each document.

### Assignment Names are Distinct

Idenfiers which are nested don't refer to the same name.

Here `a` refers to different names in each line:
```
a = 0
x = (a = 1)
y = (t = 2, u = (a = 3), v = 4)
```

### Nesting Inline Strings

Whenever a String is inline there are a few rules to make it less confusing:
* More number quotes always escape fewer. Additionally, you can't embed more quotes within fewer quotes inline.
  ```
  ""a "b" c"" // Works b/c " is within ""
  "a ""b"" c" // Syntax error b/c "" is within "
  ```
* Can't put multiple quotes next to each other b/c it looks like repeated quotes w/o a delimiter.
  ```
  "a", "b", "c" // Works
  "a""b""c"     // Syntax error
  ```

Even with these rules nesting inline Strings can still require a bit of manual tracking.

Here are some of the cases which are valid but visually noisy:
```
""a "b"""   // Appears like ends with `"""` but is really a nested quotes (`"` in a `""`)
""a \"b\""" // Appears like ends with `"""` but is really a escaped quote `\"` near the closing quotes `""`
"a \"b\""   // Appears like ends with `""` but is really a escaped quote `\"` near the closing quotes `"`
```

Each of these cases can be made more clear using a `Verbatim String`, `Block StringLiteral`, or `Block Verbatim String` like:
```
// Verbatim String
'a "b"'

// Block StringLiteral
"""
  a "b"

// Block Verbatim String
'''
  a "b"
```

### Degenerate Records

Since names are optional, records naturally degenerate into a sequence of values:
```
rec  = (a = 1, b = 2, c = 3)
list = (1, 2, 3)
```

Notice the values of `rec` and `list` are the same but `rec` has names.

This will be used by higher syntax levels to interpret `Records` in different ways.

### Terminate Inline

Most syntax terminates inline.
This means there's not always a reason to do the manual work of closing elements when not needed.

Here are a few examples:
```
a = (1, 2, 3             // record implicitly closed
b = ((1, 2, 3), (4, 5, 6 // must close first record but next and outer parens will be implicitly close
c = "1 2 3               // quoting is implicitly closed
d = ""1 2 3 "4 5 6       // quoting is implicitly closed
e = ("1 2 3", "4 5 6     // must close first quote but next quote and outer parent will implicitly close
```

The only syntax which don't terminate inline are Block Strings when used in Block mode:
```
// Block Strings in Block mode terminate on DEDENT
str-0 = """
  a b c

str-1 = '''
  a b c

// Block Strings in Inline mode terminate on \n
str-2 = """a b c

str-3 = '''a b c
```

As functions, data, and expressions nest this becomes more helpful and saves developers from drugery.


## Design Decisions
------------------------------------------------------------------------------------------------------------

Some design decisions were difficult to construct a reasonable solution.

This is all the more challenging because Tozen syntax will grow in scope and complexity.
Simpler syntax design choices must be compatible when it needs to be extended.

Here are some of the justifications for our design choices that were challenging.

### Why 2 space indents?

Often the issues with whitespace indentation are caused by:
* different assumptions (tabs, number of spaces, etc.) used across files
* developer preferences, usability, and consistency in their own source
* different tools handle whitespace differently and inconsistenly

These issues conflict between the goal of sharing code across developers with the developers own experience.

The formal requirement for Tozen is that prefixed whitespace uses 2 space indentation saved to the file.
However we will endevour to create tools to ameliorate this by making leading whitespace visualization customizable.
The lexer and parser will have mandatory utilities to specify file leading whitespace in whatever units the developer desires while assuring they retain the formal requirements when saved.

Hopefully this is a balance which acceptably ameliorates the conflicting issues and goals.

### Upper-Case Identifiers are Special

Upper-Case identifiers indicate they are processed different than other identifiers.

At this syntax level, they are used to indicate special values (like `None` and `True`).

At higher levels these will expand to include other concepts like `Type` and `Keyword`.

### Why precedence of `=` less than `,`

It was a difficult decision to decide the precedence of `=`.

Ultimately it was chosen b/c:
* Able to enumerate multiple names and values inline
  ```
  width = 100, height = 20, x = 0, y = 150, z = 300
  ```
  is unambiguously intepreted as:
  ```
  width = 100
  height = 20
  x = 0
  y = 150
  z = 300
  ```
* Assigning multiple values becomes more clear as multiple values must be within parentheses
  ```
  x = (1, 2, 3)
  ```
* Values are more important than names.
  The priority should be to focus on values.
  It is more common to pass a series of values (w/o names) than names (w/o values).
* More levels of delimiter precedence will be added later to accomidate any gaps

### Why `Record`-like insted of `List`-like?

Often languages which use data for syntax use a `List`-like core data structure.

My choice is nearly the same in that:
* values are given priority over names
* names are optional
* `Records` naturally degenerate into `List`-like structures

Although allowing optional names provides some utility:
* A more natural syntax for assigment and destructure operations
* Easier to merge assignment and attribute description syntax 
* Names can be added for zero computation cost and a minimal amount of added complexity

### Prioritizing Names

When designing a language there are many different naming layout conventions that can be selected from.
One commonly chosen is to start declarations using qualifiers, (eg `const`, `volatile`), modifiers (eg `long`, `short`), keywords (eg `volatile`, `auto`, `static`), types, or many others before a name.

To correctly model a problem developers should be able to add as many constraints as they need.
Since these can be numerous, we take an alternate approach and put the name first.

Names starting the line is easier for understanding the structure, shape, and flow of code.
Modifiers, constraints, and other syntax for declarations are significant as well but we've created alternative syntax to describe them.
Also allows developers to feel free to add as many constraints as the problem needs with minimal loss in the flow of code.

This choice has a few effects like:
* Names almost always start a line
  * This make data-definition and L-hand assignment consistent which is very common
  * It makes it easier to browse and grok the shape of the data or block the developer is looking at
  * It's easier to align elements into logical units where names are on the left, values are on the right, and how they relate to one another is in the middle (often assignment `=`)
  * The syntax is more regular between data declaration, definition, usage, and description
* We don't have special syntax, modifiers, or operators before or surrounding names.
  Instead the operators, relators, and values are used to specify these.
  * Generally names and relators describe where, values describe what, and relators describe how.
* As Tozen is used in different domains, it will become more common for developers to introduce their own data, types, keywords, and more.
  Requiring a consistent pattern of name then relation (often `=`), then value provides some consistency.
* Since Tozen syntax can be used in different paradigms (eg declarative, logic, and procedural to name a few) it is more consistent
  * This is because the additional information, rules, fields, etc. will always follow after the name
* Names act like their own namespace where their attributes, metadata, and other details are described within it.
  * Defining a context and then it's details within it will become a common pattern.
    Names first make this even more consistent.


## Summary
------------------------------------------------------------------------------------------------------------

This document describes the lowest-level, opinionated syntax of Tozen without any sugar.
This is a useful b/c it allows for developers to program and generate programs with a very minimal and consistent syntax.
This gives us many of the benefits of a S-expression like syntax with a few sacrifices.

The next document will introduce sugar to make this low-level syntax more ergonomic for developers.

### Notation

All of the notation defined so far:
```
,         // Delimiter
()        // Grouping for explicit precedence
=         // Name to Value Operator
//        // Comment
//-       // Document String
''        // Verbatim String Literal
""        // String Literal (supports escaping)
-         // Negative Unary Operator
\         // Escape and line continuation
.         // Object-Property relator
:         // Data-Metadata relator
/         // Parent-Child relator
True      // Boolean Value Literal
False     // Boolean Value Literal
None      // No Value Literal
```

### Precedence

The precedence table so far is relatively simple (high to low):
* Parentheses
  * eg `x = (1, 2, 3)`
* Prefixed, Unary Operator
  * eg `-4`, `+4`
* Infix Assign
  * eg `x = 1 2 3`
* Comma Delimiter
  * eg `x = 1, 2, 3`
