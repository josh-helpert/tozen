* Q
  * How to deal w/ spaces in identifiers and values?
    * Shouldn't conflict w/ function call syntax sugar?
      ```
      f(x, y = 1, z)
      f x, y = 1, z
      f (x, y = 1, z)
      ```
  * Literal grouping which consider a single lexical or semantic unit
    * Interpret in consistent way for common pattern and avoid noise?
  * Define vs Query vs Overwrite
    * How to avoid mutation in some cases? Just use the latest value in order?
      ```
      x =
        a = 0
        b = 1
        c = 2

      x.a = 3

      x.(a, c)

      x.(a, c) = (4, 5)

      y =
        x.b = 6

      z:
        x.b = 7
      ```
    * No syntactical difference between definition and mutation just which is defined first and in which scope?
      * By default, there are no scopes and nesting implies relation (name-value, parent-child, or other relator)?
  * How should we deal w/ adding keys dynamically in terms of syntax?
    * Don't we ever want to evaluate a key from a scope and then set it?
      ```
      test = 'abc'
      m = (`\(test)` = test) // m = (abc = 'abc') ???
      ```
  * 

## Relators

### Syntax Forms
```
// Inline form
o.(a = 1, b = 2, c = 3):(d = 4, e = 5)/(f = 6, g = 7, h = 8) = 0

or // Could be useful to avoid chaining and just use SUGAR instead of complex rules?

o .(a = 1, b = 2, c = 3) :(d = 4, e = 5) /(f = 6, g = 7, h = 8) = 0

// Since relators are traversal/queries over data when inline should only be used for select?
o.(a, b, c):(d, e)/(f, g, h) = 1..8
```
* Inline relators can chain together to indicate they both target the same name.

### Record-Relator Equivalence

```
// Inline
r/(a = 1, b/(c = 2, 3, 4, d/(e = 5), f = 6), g = 7)

or 

r /(a = 1, b/(c = 2, 3, 4, d/(e = 5), f = 6), g = 7)
```

Rules:
* Atom or Record
  * atom: `a = 1`
  * record: `a = (1, 2, 3)`
    * what about `a = (b = 1)`? should be `a/b = 1`? thus a record?
* Indentation uses `/` relator by default

Notice that:
* Relator punctuation identifies a namespace
* Each relator child is a field
  * A field is a descriptor under that relationship
    * object - descriptor-type - descriptor - value
    * A `car` `property` of `color` is `Red`
    * car <property = color> 'Red'
  * A field is a Record which can have it's own value and relators
* 

Notes:
* Record and Relator have equivalence (which relate to sugar too?)
  ```
  // Record syntaxes
  r = (t = (x = 0))

  r =
    t =
      x = 0

  // Relator syntaxes
  r/t/x = 0

  r
    /
      t
        /
          x = 0

  r
    t
      x = 0
  ```
  * Records are just parent-child relator
    ```
    // Record syntaxes
    r = (t = (a = 0, b = 1, c = 2))

    r =
      t =
        a = 0
        b = 1
        c = 2

    // Relator syntaxes
    r/t/(a = 0, b = 1, c = 2)

    r
      /
        t
          /
            a = 0
            b = 1
            c = 2

    r
      t
        a = 0
        b = 1
        c = 2
    ```
* Record and Relator allow for sugar
  ```
  r = (.a = 1, :b = 2, /c = 3)

  r
    .a = 1
    :b = 2
    /c = 2
  ```
  * Should meaningless be allowed like:
    ```
    r =
      .a = 1
      :b = 2
      /c = 2
    ```
    * Should act just-like sugar of `r = (.a = 1, :b = 2, /c = 3)`
  * What about mixing values and relators looks weird?
    ```
    r = (a = 1, b = 2, c = 3, .d = 4, :e = 5)

    r = (/a = 1, /b = 2, /c = 3, .d = 4, :e = 5) // Require `/` when mixed?

    r = (a = 1, b = 2, c = 3)
      .d = 4
      :e = 5

    r =
      a = 1
      b = 2
      c = 3
      .d = 4
      :e = 5
    ```
    * Default context is Record
      * Only by specifying other values (eg Integral) does one change?
    * Most relators (`.` and `:`) target this node and value while `/` targets children
      * So a bit different?

### Relator and Value Merge

```
// as a Relator
my-rec/(a = 1, b = 2, c = 3, d = 4, e = 5)

or? my-rec /(a = 1, b = 2, c = 3, d = 4, e = 5)
```

### Values are Context

Notes:
* Values give the context that relators are interpreted in
  * eg
    * `x = (1, 2, 3)` => ctx is Record
    * `x = [1, 2, 3]` => ctx is Array
    * `x = 7`         => ctx is Integral
  * 
* Refinement vs equal syntax
  * Relators refine a name
    * Refinement from under name
  * Records are values
* Mix relators and records
  ```
  (x, y, z) = (a = 1, b = 2, c = 3)
    .t = 4
    .u = 5
    :v = 6
  ```
  * Do the properties apply to each element b/c a destructure? or invalid statement?

```
// Actually default is ` ` b/c there's not context to interpret it in?
// - And that just happens to be a `Record`?
// - Instead of using `None`? what does `None` imply?
r =
  a = 1
  b = 2
  c = 3
```

### Name is Root

For both values and relators the name is the root namespace.

Values are special because they're the default relationship to the name and have the least amount of syntax.
They work like most languages.

Relators are not the default relationship to a name.
Instead, we must indicate it using `.`, `:`, or `/`.

It can be useful to view names as a tree root with different namespaces for each relationship:
```
r                  // Root Name
  = 4              // Space of Values
  .mem-width = 1   // Space of Properties
  :type      = Int // Space of Metadata
  /child     = 7   // Space of children
```

### Refine

Although relators are very similar to records they don't function the same.

Since relators are like a namespace they operate on the name they wish to modify.


### Orthogonal

TODO:
* Value is context
  * props and metadata are applied to value
* Value and relators are all diff aspects of the name
* How to handle destructure? make illegal?

Values are relators are similar in that:
* they can be envisioned as a type of namespace containing records
* branch off of the name which acts as the root of each space
* have a semantic meaning associated to them:
  * `=` indicates the value space
  * `.` indicates the property space
  * `:` indicates the metadata space
  * `/` indicates the child space

### Refinement

TODO:
* relators don't use `=` and not a value
  * instead just a shorthand to avoid common errors?
* `=` isn't always meaningful
  ```
  r =
    .x = 1
    .y = 2
    :z = 3

  =>
    r = Record ()
      .x = 1
      .y = 1
      :z = 3
  ```
* relators as a value (not under a name) is just a convenience
  ```
  (.x = 1, .y = 2, :z = 3)
    => _.(x = 1, y = 2):(z = 3)
    => ().(x = 1, y = 2):(z = 3)
    => None.(x = 1, y = 2):(z = 3)
    => Empty.(x = 1, y = 2):(z = 3)
  ```
  or
  ```
  r = (.x = 1, .y = 2, :z = 3)
    => r.(x = 1, y = 2):(z = 3)
    => r
        .(x = 1, y = 2)
        :(z = 3)
  ```

### Refinement

Notes
* Diff between `=` and just indentation
* Semantic meaning of refinement vs values?
* 


### Notes

The mechanism pairs multiple elements:
* Punctuation
* Relation Type
* Syntax Patterns
* Mimic Structural/Shape
* 



Notes:
* Relators are spaces of a specific relation paired w/ punctuation
  ```
  . property-space
  / child-space
  : meta-space
  # comptime-space
  = value-space different but is a space?
  ```
  * Spaces under a name which describe types of relations (aspects/lenses) of that name
* Getting values
  * Name gets natural value (eg `x` returns natural value of `x`)
  * Relator must be requested (eg `x/a` returns natural values of `a` childs of `x`)
* Paths
  * Definition using exactly shape of data (almost literate)
  * Query very similar as would expect
* Values use Relators as well
  * Relators act like upsert:
    * if used by value, nests w/n value?
    * if not used by value, nests w/n name?
  * common case is Atomic vs Composite
    * if Atomic    => `/` under name
    * if Composite => `/` under value
* 
* 

```
r.(a = 1, b = 2, c = 3)

r = None
  .
    a = 1
    b = 2
    c = 3

r =
  .
    a = 1
    b = 2
    c = 3
```

```
r/s.t:(a = 1, b = 2, c = 3)

r = None
  /
    s = None
      .
        t = None
          :
            a = 1
            b = 2
            c = 3

r = None
  s = None
    .
      t = None
        :
          a = 1
          b = 2
          c = 3

r
  s
    .
      t
        :
          a = 1
          b = 2
          c = 3
```


## Relator Forms
------------------------------------------------------------------------------------------------------------

Notes:
* Should we allow for value to have elements and for children as well?
  * Benefits
    * Don't have to create multiple layers just to model value of children and then other children
      * Like some type of grouping?
      * Similar issue if value conflicts with any relator?
  * Downsides
    * Out-of-bound modeling where value can be anything that relators can be
      * Instead all should be modeled w/ known structures?
* 

```
r =
  
```

## Working Set
------------------------------------------------------------------------------------------------------------

* Notation
  ```
  ,         // Delimiter
  ()        // Grouping
  =         // Name to Value Operator
  .         // Object-Property Relator
  /         // Parent-Child Relator or Operator Division
  :         // Data-Metadata Relator
  //        // Comment
  True      // Boolean Value
  False     // Boolean Value
  ""        // 
  ''        // 
  ``        // 
  -         // Unary Negative Operator or Anon Elem
  ???       // Placeholder?
  _         // None sugar?
  \         // Escape
  None      // No Value
  Undefined // Unbound Name
  ```
* Delimiters
  * Only `,` for min?
* Precedence
  * Write out global precedence table? just a few characters
  ```
  ()        // Grouping
  -         // Unary Operator
  /         // Binary Operator
  =         // Name to Value Operator
  ,         // Delimiter
  ```
* Others?
  * Literals? or these for future and reductive?
    * what are the min set of literals?
    * should only use literals as the most universal set which are not ambiguous in nearly all cases
      * then we handle edge-cases by:
        * importing/changing inference extensions which locally expand the inference rules
        * using explicit inference/code ``
        * others?
  * Reserved
    ```
    #
    if
    loop
    do
    Type
    :type
    ;
    |
    []
    {}
    ```
  * Relator defaults for spaces on newline?
    * should just be child but indicate it's definable?
  * spacing for identifiers? and names? and values?
    * spacing in general needs to be described?
    * as well as what is valid identifier?
  * `x = \n`, `x \n =` and other combinations?
  * type of quoting ("" '' `` and their uses)
    * use-cases to handle
      * Literal
        * inference even work here b/c how to differentiate `f x` between functions (`f(x)`)and a single name (`"f x"`)?
          * would have look in scope which isn't ideal either
        * What can be used to group identifiers together? and differentiate issues like this?
        * The only usefulness of a literal is when paired w/ the future `:` operator so we can automatically infer?
          * Even in that case how do we determine functions? or just assume all non-Numeric and non-Boolean must be string literals?
          * We can ignore literals for now and just use a special mechanism for `:` or there really a need for literals outside of it?
      * String (Escapable vs Verbatim)
        * Could also use `"""` and similar to imply verbatim?
          * Then how would we do block strings which also need to have escapes? post-processing by pipe at comptime?
        * Would it even make sense to force escapable strings *must have* an escaped value else must be verbatim? So we know what we're looking for and guarantees?
          * A strong neough guarantee and big enough problem to care about?
        * Additionally differentiating verbatim from escapable string would mean that we have a trivial way to use escape (`\`)?
      * Code
        * Which degenerates into literal when unambiguous?
      * 
    * `` for code
      * Could also use "" and then have added refinement for code?
      * Is code just a literal group? and and reduce to known b/c will be lexed/parsed?
    * ""
    * ''
  * Examples
* 

### Multiple Paths

Paths and selection are not limited to the the same parent.

The trick of doing this consistently is that when multiple selections across paths are used it creates a single, flattened sequence.

Is this true? what issues and edge-cases?

```
// Assign both names `x, z` to values `5, 6`
o.b:(x, z) = (7, 8)

// Translates to `(x, z, t, u) = (7, 8, 9, 10)`
(o.b:(x, z), o.c:(t, u)) = (7, 8, 9, 10)
```

```
o
  .a = 0
  .b
    :x = 1
    :y = 2
    :z = 3
  .c = 4
    :t = 5
    :u = 6

o.(a, b:(x, y ,z), c:(t, u)) => 
  o.a, o.b:x, o.b:y, o.b:z, o.c:t, o.c:u
  (o.a, o.b:x, o.b:y, o.b:z, o.c:t, o.c:u)
  (o.a, (o.b:x, o.b:y, o.b:z), (o.c:t, o.c:u))

// Stuff like this works as just a convenience?
o.b:(x, z) = (7, 8)
// but is really b/c parens drop?
o.b:x, o.b:z = (7, 8)
// so we don't have to do this:
(o.b:(x, z)) = (7, 8)
```




### Literal Values

* Singular
  * Undefined
    * Represents initial value not yet defined
    * Not usable until provably defined
    * Only value of type `Undefined` is value `Undefined`
    * Not castable to other types (like `NULL` in most languages)

## Issues, Edge-Cases, Ambiguities, and Quirks
------------------------------------------------------------------------------------------------------------

### Spaces (key and value)

```
k0 k1 = v0 v1 v2
  `k0 k1` = `v0 v1 v2`
  k0, k1 = (v0, v1, v2)
  k0, (k1 = v0), v1, v2
```

### Nesting Inline Strings

* How to handle?
  ```
  "a" "b" "c"
    => "a b c"
    => "a", "b", "c"

### Record Names are Optional

Using `=` is what differentiates a key from a value.

Consider a simple record without keys `r = (1, 2, 3)`.

When we add in any keys to the record 

### Pure Syntax Meaning

* only applicable to Record syntax
* only becomes data when is a type which has a runtime cost like String or eventually Array?
* many things are erased before runtime
* 

Elder uses data structures for both it's syntax and data structures.

In the same way that LISP uses S-expressions as both syntax and data; Elder uses records.

The context records are used in determine how they're interpreted.


Records are used to model code, data, types, 

## Literals
------------------------------------------------------------------------------------------------------------

### Units?

### Literal Types

```
[]
{}
||
<>
```

## Compiler Processing Modes
------------------------------------------------------------------------------------------------------------

* skip steps
  * no lex
  * no parse
  * no whatever?
* assume is whatever shape/type?
* 


## TODO: Placeholder
------------------------------------------------------------------------------------------------------------

```
r =
  1
  a =
    2
    #?  // Implies subform should always be considered valid
      t = x
      u = y
      v = z
    3
  4
```

Ideas:
* Other cases
  * until end of line `#?` like triple quoting?
  * inline `#?( ... )`


## TODO
------------------------------------------------------------------------------------------------------------

* Better names for String
  * we need to understand literal vs verbatim vs escapable vs ...
* what should be special-cased?
  * Verbatim single characters? eg NULL terminitaor?
    ```
    \n
    \r
    \t
    \0
    ```
* priming? `x x' x'' x'''`?
  * ambiguous w/ verbatim?
  * other ideas
    ```
    x^^^ == x^3 // superscript-like
    x___ == x_3 // subscript-like
    ```
* line continue spacing is invalid
  * depending where embedded in StringLiteral or code (generate delimiter?)
* when is syntax like this allowed?
  ```
  a = (b = 1) // 
  c = a       // is not allowed currently but eventually will be?
  ```
* units
  ```
  42_sec
  42__sec
  42#sec
  42##sec
  42_sec-per-mile
  ```
* Pipe `|` is marker for operations
  * eg piping steps
  * eg for split grouping names to groups of values
* casts
  * really 2 issues for casting:
    * some casts are not meaningful
      * eg Int -> Bool
      * eg Person -> Bool
    * information change
      * eg Int to Float changes the value due to it's representation
    * information loss
      * eg U16 to U8 drops 8 bits

## Destructure

### Simple Destructure

The existing syntax can do simple destructuring.

There are 2 constraints:
* They must have equal arity
* Multiple names can't map to the same value 

Here are some examples to clarify:
```
(x, y, z)   = 1              // Illegal b/c different arity
(x, y, z)   = (1, 2)         // Illegal b/c different arity
(x, y, z)   = (1, 2, 3)      // Works
(x, (y, z)) = (1, (2, 3))    // Works
(x, (y, z)) = ((1, 2), 3)    // Illegal b/c `(y, z)` both map to `3`
(x, y, z)   = (1, 2, 3, 4)   // Illegal b/c different arity
(x, y, z)   = (1, 2, (3, 4)) // Works b/c now element arity matches which causes: `z = (3, 4)`
```

In higher layers destructuring will become more flexible as sugar and utilities are added.
However it all reduces to this simple form.

### Simple Destructure with Relators

Although the core syntax can express simple destructuring it is illegal to mix destructuring with relators.

This is because it's unclear which names to apply the relators:
```
(x, y, z) = (a = 1, b = 2, c = 3)
  .t = 4
  .u = 5
  :v = 6
```

Instead of special cases we just don't allow for it.
If a developer wants to use relators with destructuring they will have to do so explicitly for each binding.
