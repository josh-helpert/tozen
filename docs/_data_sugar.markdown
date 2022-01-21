
## Assignment Repitition for Precedence
------------------------------------------------------------------------------------------------------------

Notes:
* Only `==` for destructure?
  * Makes some sense b/c `:` is sugar for mapping identifiers to `primitive types`?
* What does it mean for `===`?
  * mapping to the entire line on L and R hand?
    ```
    a b, c; d | e === 1 | 2; 3, 4 5
    ```
* Precedence
  ```
  Ops       | Terminates
  ---------------------------
  = :       | , or greater
  == ::     | ; or greater
  === :::   | \n
  ```
  or:
  ```
  Ops       | Terminates
  ---------------------------
  = :       | , or greater
  == ::     | ; or greater and pick
  === :::   | ; or greater and structural
  ```
* 

Destructure vs this? `==` or `::` like:
* `x, y, z == 1, 2, 3`
  => `(x, y, z) = (1, 2, 3)`
     `x = 1, y = 2, z = 3`
* `x, y, z :: 1, 2, 3`
  => `x, y, (z = "1, 2, 3")`
  => `x = 1, y = 2, z = 3`

```
m :  a b c
  => "a b c"
  => "a, "b", "c"

m :: 4.6, Jill Jane, 27 inches
  => "4.6, Jill Jane, 27 inches"
  => 4.6, "Jill Jane", "27 inches"
```

```
x = {...}
x:y.z{...}
x:y.{...}
f{...}
o.(a, {b, c}, d)
```

```
x:y.(a = 0, b = 1, c = 2)

x/y/z/(a = 0, b = 1, c = 2)
```

## Relators

### Inline Definition Syntax

Notes:
* Not able to nest by format of syntax (gets to complex)
* With spaces will always define
* Delimits without spaces
* Must use parens?
  * certainly makes it cleaner to read
  * must group together when same relator?
    * what is a nested path?
      ```
      x .(a = 1) .b:c/(d = 2) = 0 // wrong?
      x .(a = 1, b:c/d = 2)   = 0 // right?
      ```
* how to deal w/ spaces and names?
  ```
  x .(1, 2, 3) // values?
    => x .(_ = 1, _ = 2, _ = 3)
  ```
* can mix between names and values b/c they're orthogonal?
  * not exactly b/c when execution it will get the value? `x .(1, b, c) => lookup value of 'b'?`
* issues with nesting? allowed?
* 

### Chain Form

In order to assign both values and relators inline we need a syntax which captures this pattern.

Consider the a multiline version which assigns both values and relators:
```
x = 0
  .a = 1
  .b = 2
  .c = 3
```

To do the same inline we append the relators to the name:
```
x.(a = 1, b = 2, c = 3) = 0
```

When describing a nested name you can just use the path directly like:
```
x.y.z = 0
  .a = 1
  .b = 2
  .c = 3

// is equivalent to:
x.y.z.(a = 1, b = 2, c = 3) = 0
```

Can even assign multiple names and relators:
```
o = 0
  .t = 1
  .u = 2
    .x = 3
    .y = 4
      .a = 5
      .b = 6
      .c = 7
    .z = 8
  .v = 9

// is equivalent to:
o.(t = 1, u.(x = 3, y.(a = 5, b = 6, c = 7) = 4, z = 8), v = 9) = 0
```

Names


Differentiate Cases?
* this means we could use numbers (and other) for keys b/c they're static?
  * it would also mean we need to query/select them...they must both be allowable and clear!
* cases
  * define/select (L-hand)
    ```
    ```
  * value/query (R-hand)
    ```
    ```

## Inline Defn or Query
------------------------------------------------------------------------------------------------------------

Notes
* have a mixed syntax which allows for both and then 2 which are specifically for defn or query?
  * having weird combinations and more forms can cause issues if there's no utility from them
* what issues are with composing? not all can compose and not all should compose
* path vs query vs define?
  * query can use path as just a query along that path to those names?
    * but other queries could be created as well?
    * path and variable itself (path to a single value) don't require any special syntax?
  * if we can know what is def and what is query we can have sugar for definition?
    * ie we know if something is an identifier b/c it's just a string w/o literals?
* must be true
  ```
  x = 4
  x = y
  x.y = 4 // path
  x:type = Int
  x:(type) = 4 // allowed? select or define? similar to x.(a, b) = 1, 2? or unique?
  x:(type = Int) = 4 // internal def but not exactly path? maybe this uses special syntax to emit name?
  x.y/z:(type = Int) = 4 // path and internal def
  p.(height = 20px)
    span = My
    span = header
    span = is
    span = long
  ```
  * these patterns imply:
    * 

```
x = (.a = 1, :b = 2)

?x.(a, b)
?x.(1, 2)
?x.(#1, #2)

@x.(a, b)
@x.(1, 2)
@x.(#1, #2)

x.(a, b)
x.(1, 2)
x.(#1, #2)

x.(a, b) = 1, 2

(x.a = 1, x.b = 2)

x.(a = 1, b = 2)

x.(a = 1, b = 2) = 0

x = 0.(a = 1, b = 2)

x.(a = 1, b = 2) = 0

x.(a = 1, b = 2) = 0.(c = 3, d = 4)

r = /(a = 1, b = 2, c = 3) = (a = 1, b = 2, c = 3) ?

(x.(a = 1), y.(b = 2), z.(c = 3)) = (4, 5, 6) ? desirable? or should be separate lines?
```

## Section
------------------------------------------------------------------------------------------------------------

Ideas:
* Sections are vertical indicators of a step
  * They make it easier to follow the flow and structure of the code
    * Much like indentation are indicators of blocks for scope/siblings
* Needs to have a way to embed rich media in documentation blocks
  * Like an inverse literate program where the comments and docs are rich text and can better inform the flow?
* Questions
  * How to compose different categories w/o confusing their uses?
    * If they're visually distinct we could make it clear which are executing and which are docs and whatever else?
  * Just visual indication of other utils like comptime functions and macros?
    * Not only b/c some is semantic structure
      * even some semantic structure to be processed in specific way like `### import`?
    * eg modifiers
      ```
      ### public, const
      x = 1
      y = 2
      z = 3

      is

      x = #public #const 1
      y = #public #const 2
      z = #public #const 3
      ```
* Aspects
  * role and interpretation
  * data representation in Elder base
  * names (in/out)
  * configuration
    * metadata
    * rules
    * knowns
    * modifiers
    * patterns in syntax and others
      * inline syntax templates?
  * syntax and sugar
  * nameless
  * how does it pair and jive w/:
    * `#shadow`, `#use`, `#only` and others?
  * scope?
* Different purposes for steps/sections
  * ReStructured Text
    * Role
      * Inline semantic markup
      * eg Area of cirle is :mapth:`(pi / 4) * d ^ 2`.
    * Directive
      * Block of explicit markdup
      * eg
        ```
        .. image:: bio.png
          :width:  200
          :height: 100
          :scale:  50
          :alt:    Bio Image
        ```
    * Domain
      * Collection of Roles and Directive that belong together
      * eg
        ```
        .. c:var:: int 1 = 42
        .. c:function:: int f (int i)
        ```
  * role/mode
    * a la ReStructured Text
    * changes interpretation?
      * eg `#lang/JavaScript`?
  * steps
    * build up computation in steps
      * should these be orthogonal to blocks?
        * or have another concept which allows composes both of them?
  * branch
  * context/domain w/ keywords
    * each context can have keywords which can be their own context (to nest)
      * keywords just organized vertical structure as section
      * keywords are just names w/ apply and can be joined
      * keywords can filter and conditionally apply w/ a predicate?
    * eg vars
      ```
      ### var
      ###### public and API

      count:Int = 0
      is-on:Bool = True

      ###### private

      is-signed-in:Bool = False
      ```
  * pattern/template
    * eg access modifiers
      ```
      ### public
        access = RW

      x:Int  = 7
      y:Bool = False

      ### private
        access = None

      z:Int = 3
      ```
  * logical context (alike things)
    * logically grouped together
    * similar to restructuredtext-like domain?
  * exactly same (vs similar)
    * exactly the same things (w/ a predicate) could be useful as it's a very strong guarantee?
      * maybe even something like refinement and increasing specificity?
      * a way to DRY up a pattern as well and apply complex rules
        * could also filter and apply w/ a predicate?
  * context
    * use to structure info
    * semantic units
      * sometimes processed and sometimes just structural
    * elements
      * sub-context
        * used to nest context but not execute
      * keywords
        * some type of executation context
    * how to differentiate information which will execute or not?
      * should these have different prefixes or keyword() or something?
  * knowns
  * same rules
  * different interpretations of body
  * only descriptive and metadata w/o affect on code
    * eg parts of a document
  * tagging
    * imports and exports
    * allows for custom processing by compiler
* Factors
  * Isolated or not
    * Isolated: eg Block, Computation, Step
    * Not:      eg Modifier, Descriptive, Alias, 
  * Terminates On
    * End of Scope/Sibling
    * Next Section
    * Dedent
  * All, Some, or None are same
    * All: 
    * Some: 
    * None: 
  * Process or not
    * Process: Modifier, 
    * Not:     Comment, Doc, 
  * Orthogonal or Not? or what?
  * 
* Cases
  * Steps of Computation
    * elements: doc string  +  Block/Isolation  +  In/Out
    * a series of computational block which have documentation as to why each step
    * how does this compose w/ nested levels of steps? 
      * also how does it compose w/ other types of cases?
  * Section (but not block?)
    * eg UI vs Logic vs Style
    * similar to Section and Transition (but w/o title)
      * Transitions are more about transition between Body elements and not Sections?
        * May not be useful concept for code? instead of a narrative?
    * a consideration is when is isolated and when isn't?
      * should isolated be another concept/aspect?
    * potentially just a relator and have a way to make it visually clear?
      * eg web framework
        ```
        ### UI
        ###### .style

        width = 100%
        height = 20px

        ### Logic
        ####### Query

        body   = $('body')
        header = $('.header')

        ####### API
        ...
        ```
  * Keywords/Trigger
    * filter and apply
    * tags for custom processing 
    * does not introduce a `Block`
      * instead it processes each of the elements and returns the results
      * need to assure it terminates in places that make sense?
        * sometimes we need to terminate on DEDENT, INDENT and/or SIBLING?
          * on DEDENT
          * on INDENT
          * on SIBLING
    * eg `public`, `private`, `import`, `export`
    * a way to process all the children w/n the keyword
      * often changes based on type of child
      * so very much like a filter processing
  * Foreign/External
    * not processed and considered foreign code
    * can pass args to foreign block
    * possible to emite output data and results?
      * if there's some type of external evaluation?
    * eg `#lang/JavaScript`
  * Directive
    * eg `Type` interpret lexeme
  * Doc
    * For developer and for computer processing, analysis, generation
  * Comment
    * For developer only
* Syntax
  * `###` comptime indicator
    * notes
      * config w/o added prefix so don't have to manually add it each time
        * also immediately after the indicator which also forces a gap between indicator and body (whether present or not)
      * can be in optional and required mode
      * not ideal b/c configuration can't have newlines!
        * maybe better if we a diff way to identify config vs body?
    * examples
      * Continue Section / Rest of siblings/record
        ```
        ### {optional-title}
          {optional-config}
        \n
        {body}
        ```
      * Block Section
        ```
        ### {optional-title}
          {optional-config}
        \n
          {body}
        ```
      * Explicit config
        ```
        ### {optional-title}
          #config             // acts like a keyword?
            {optional-config}
        \n
          {body}
        ```
        * Gets around issues w/ newlines in config?
  * Computation Steps Ideal Flow (a special sequence)
    ```
    //--- step-0 doc ---
    input-names
      ...
      body
      ...
      results

    output-names // Don't have to look at start (and may need explanation?)

    or:

    //--- step-1 doc ---
    input-names
      ...
      body
      ...
      results

      output-names
    ```
  * Meta-syntax pattern which adds in alternative structure?
    * eg docstring `///` but can apply structure to meta-patterns by:
      ```
      // Maybe only allow this for top-level?
      //---
      // Level 0
      //---

      //- Level 1 ---
      //--- Level 2 ---
      //----- Level 3 ---
      //------- Level 4 ---

      //- Level 1
      //--- Level 2
      //----- Level 3
      //------- Level 4

      //---    // Terminal
      //---/// // Terminal
      ```
    * eg structure `###`
      ```
      // Maybe only allow this for top-level?
      ##---
      ## Level 0
      ##---

      ##- Level 1 ---
      ##--- Level 2 ---
      ##----- Level 3 ---
      ##------- Level 4 ---

      ##- Level 1
      ##--- Level 2
      ##----- Level 3
      ##------- Level 4

      ##---   // Terminal
      ##---## // Terminal
      ```
  * Patterns
    ```
    ===
    Top-Level
    ===

    ---
    Sub-Section
    ---

    ### a
    === a
    *** a
    +++ a
    ^^^ a
    --- a
    ___ a
    ... a
    ```
* Examples
  * 
* Definition Syntax
  ```
  o .(a = 1, 2, b = 3) /(c = 4, 5, d = 6)

  ===

  o
    .a = 1
    ._ = 2 (or `.(2)` or `.None = 2` or `.(= 2)`)
    .b = 3
    c = 4
    5
    d = 6

  ===

  o
    .
      a = 1
      2
      b = 3
    c = 4
    5
    d = 6

  ===

  o
    .
      a = 1
      2
      b = 3
    /
      c = 4
      5
      d = 6
  ```


## Relator
------------------------------------------------------------------------------------------------------------

Relators are a syntax abstraction which merges a namespace with a type of relationship.

It describes patterns which other languages are often use hard coded syntax.
Here's a few examples:
* `C++`
  * accessing a `Struct` field uses `.` syntax like `my_stuct.field` or `my_struct.field = 1`
  * accessing a `Namespace` uses `::` syntax like `my_namespace::my_val`
* `Clojure`
  * accessing a `Namespace` uses `/` syntax like `my_namespace/my_val`
  * interop with `Java` or `JavaScript` uses `.` syntax like `(.toUpperCase "jane")`
* `XML`
  * accessing a child element uses `/` syntax like `car/wheel/rim`
  * accessing a attribute uses `.` syntax like `car.weight`

Elder syntax takes a different approach by allowing the developer to create their own syntax in a very constrainted way.
Customizing the syntax allows developers to add structure to the syntax without the need of macros.

To accomplish this, we introduce a new syntax we've termed a `Relator`.
Specifically a `Relator` is defined as a syntactical tool which describes the `relation` between a `object` and a `descriptor`.
Each name and value under a `Relator` share the same type of relationship.
Thus the name `relator`.

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

### Relator as Value

Although relators contain values, they aren't values themselves.


* Record and Relator allow for sugar
  ```
  r = (.a = 1, :b = 2, /c = 3)

  r
    .a = 1
    :b = 2
    /c = 2
  ```

### Nameless Relator

Just like records, it's possible to create nameless relators.



Forms:
```
obj
  .a = 0

obj
  .
    a = 0

obj.
  a = 0
```

Ideas:
```
o = (1)  = 1  ?
  vs
o = /(1) = /1 ?
  vs
o = .(1) = .1 ?

o._ = 1
o.(1)
o.(= 1)
o.(_ = 2)
```


## TODO Sugar
------------------------------------------------------------------------------------------------------------

* Relators
  * Explain why don't need `=` in relators
    * or is just optional?
  * Chaining `x.(a = 1):(b = 2, c = 3) = 0`
  * Default relator via indent
    * default indent is for parent-child using `/`
    * need to explain how to declare? or something for later?
      * only change default when doesn't use parent-child or only uses 1 type of relator?
  * Relator space declare form `o .a = 1, .b = 2, :c = 3`
  * Children slices
    * eg `o.(3 .. 5)` for children `[3, 5]` vs `o.(3 ..< 5)` for children `[3, 5)` (or `[3, 4]`)?
    * eg `o.(b .. e)` for children `[b, e]` vs `o.(b ..< e)` for children `[b, e)` (or `[b, d]`)?
    * these just ordered by known orderings or also can be the names w/n the Record b/c ordered?
  * Relator children can't conflict w/ values
  * Refinement syntax
* Need to track differences between comptime known (ie syntax) and dynamic
  * This will relate to when we access data as to whether it cost or not?
* `:`
  * Until `,` or `\n`
  * Maybe more for different precedence?
    * like `::` for `;` or higher precedence? or for later?
      * or useful for EOL?
        * operator repetition (`==`) vs mode/literal repition (`""`)
      * pairs w/ `==`?
    * how to do something which runs until `\n`? useful?
      * or an edge-case we can handle w/ `()` or some type of unit operator like `` ` ``?
    * about what is considered a `unit/grouped` vs what is considered a delimiter? core issue? one aspect?
  * escape expression?
    ```
    b = 2
    x : a \(b) c
      => x = 'a \(b) c'
      => x = "a \(b) c"
      => x = "a 2 c"
    ```
  * Determinisitic and constrained inference
    * Should match what is done for most languages?
* `{}`
  * Idea: Map-like Data
    * similar to NestedText and YAML?
    * Identifiers are implicitly converted to known literals or StringLiteral
      * can include external Schema to control interpretation rules
        * especially for those cases where many interpretations and edge-cases?
    *
* `[]`
  * If Array-like then how much more useful is it?
* Rest args for destructure?
* Elder needs layer for unconditional/branchless/total?
* Inference have a few modes?
  * modes
    * infer from the syntax itself
    * infer w/ specific constraints and ignore, collapse, merge
  * way to get benefits of LISP and dynamic languages w/ infer directly
    and
    strict typing
* Precedence
  * `Space , ; | \n \n\n`
  * How to manage multiple levels when we infer?
    ```
    l = (a b, c; d | e)
      => l = (a, b, c, d, e)       // Infer as a single level
      => l = (a b, c, d, e)        // Relative to the `=` operator
      => l = (a, (b, c, (d, (e)))) // By explicit precedence
    ```
    * Inference needs to be based on what dev expects? while staying consistent?
  * Problem is we sometimes want to use `\n` and other times we want them to merge
  * examples
    ```
    r =
      a = 1
      b = 2
      c = 3

    m =
      a    | b c     | d e
      f    | g, h, i | j
      l; m | n; o, p | q; r; s

      =>
        m =
          a, (b, c), (d, e)
          f, (g, h, i), j
          (l, m), (n, (o, p)), (q, r, s)

        m = (a, (b, c), (d, e)), (f, (g, h, i), j), ((l, m), (n, (o, p)), (q, r, s))

    // Inferrable in unknowns?
    x   = (0 1, 2 3 4, 5 6)
    x'  = Array(0 1, 2 3 4, 5 6)
    x'' = Array(Int)(0 1, 2 3 4, 5 6)

    x:type   is Record(Record(Int))
    x':type  is Array(Record(Int))
    x'':type is Array(Int)
    ```

goals:
* expressive
* terse
* consistent/predictable
* readable
* matches expectations and works like one would expect

## More Delimiters
### Inference Specificity


## Markup
------------------------------------------------------------------------------------------------------------

Notes:
* ideally a way to provide optional names which is like a title
  * this similar to modules?
    * similar but not exactly diff between *sandbox* and *module*
      * sandbox won't have title

```
---###---
###---###
###------
---|||---
```

```
---| My Title |---
```

```
## Title
------------
```


## Markup and Code Formatting
------------------------------------------------------------------------------------------------------------

* Sandbox
* 


## Syntax `{}` and `[]`
------------------------------------------------------------------------------------------------------------

Interpretation Context `{}` and `[]`

```
x = {...}
x{...}
x.{...}
x:{...}
x/{...}
```


## Example - Serialization
------------------------------------------------------------------------------------------------------------




