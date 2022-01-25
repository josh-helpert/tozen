
TODO:
* constant-time structure
  * more about what comptime can emit and analyze? then emit optimized version(s)?
* default execution environments
  * environments
    * data
    * exec (runtime or comptime)
    * other require keywords
      * 
  * mixing using keywords
    * do/#do in data
    * #data in exec
* Boolean ternary or optional?
  * True, False, Unknown
  * Result(Bool)
    * allows to include an error of not
* when is overloading grokable?
  * don't want:
    * a overload to have a non-expected implementation
    * looking overload after overload to understand what is being done
    * not all guarantees provided like:
      * + w/ commutative doesn't work w/ String?
        * or maybe this isn't a universal guarantee?
  * want:
    * avoid arbitrary additional syntax even when same semantic meaning
      * eg sum always means some type of monoid-al
    * 
  * scratch
    * the overloaded operators/functions need to follow the same semantic meaning
      * the downside is implementation details can be slightly different
        * the details may be meaningful to the context which calls them
        * for example, commutativity of `+` is important in some cases and not others
        * we can capture these guarantees and then match against the varieties of guarantees that implementations care about
          * not just guarantees but also just details, properties, qualities one would care about
            * then only the concerning cases which check the conditions which matter to them?
              * these captured as `:` metadata relator?
* More freedoms when more is known and still can be zero runtime
  * comptime known
  * local
  * once? linear?
  * mutate locally
    * not to callers
    * not to internal blocks? must be visible?
    * not until usable by any other code
      * sealed at first use in nested block?
* Variables
  * naming convention
  * modifiers
  * type specification
    * eg `x :String = "a b c"` error?
    * eg `x :VerbatimString = "a b c"` error?
    * eg `x :StringLiteral = 'a b c'` error?
  * lifetime
    * file
    * block
    * context
    * function
    * 
  * block
    * scope
      * shadowing patterns
      * #use and similar
  * comptime known
    * and how it propogates
  * narrowing
  * 
* Operators
  * only for sugar? the language only uses methods but not operators or infix outside of assignment/bind?
    * functions make more sense? and consistent?
      * eg `a + b / c` => `Add(a, Divide(b, c))`
* Precedence
* Costs
  * Not able to add in abstractions which implicitly cost (must opt-in and choose)
    * eg static (free) vs dynamic (cost) dispatch
    * eg overloading (diff args, free, comptime poly) vs overriding (parent-child, cost, runtime poly)
    * eg static dispatch (free)
      * templates in C++
      * generics
      * function overloading
      * operator overloading
      * 
    * eg dynamic dispatch (cost)
      * vtable
      * overriding
* Modules
  * look at ML, M1, and others
  * build up from simple
* Builtin Containers
  * Array
  * Struct
  * Tagged Union
  * Enum
  => what can be unified? ADT + Tagged Untion + Enum all 1 type?
* Functions
  * metadata keywords of relator in function
    * `:in`
      * `:before`
      * `:after`
    * `:out`
      * `:before`
      * `:after`
  * input
    * when by-value and when by-reference
  * stack
    * block
  * generics
  * tail recursion
  * 
  * 
  * 
* Context
* Pattern matching
* 

Notes:
* 
* 

Q:
* 
* 

## Functions
### Derived

One goal of Tozen is to move more computation into the static domain.
One way this is done is with derived computations.

### Constraints and Guarantees

At this level we aim to provide very strong guarantees by the compiler.
In order to the compiler to reason about user functions, they must conform to a number of constraints.
Although the constraints appear to be overly restrictive; a surprising amount of code fall into these constraints.

Essentially we must be able to guarantee that a function always succeeds.
Often this means it is total (each input has an output) but not exclusively.

These categories of functions can be fully reasoned about:
* comptime known: are values which are knowable by the compiler
  * often this includes:
    * `literals`
    * `const`
    * `comptime`
    * static data structures like:
      * `Array`
      * `Struct`
  * eg `sum` using comptime known
    ```
    x  = 1 // comptime known b/c literal
    #y = 2 // comptime knwon b/c comptime variable

    sum(x, #y)
    ```
* total: every input maps to some output (without error?)
  * eg static substitution
    ```
    age :U8 = 32

    msg = "user age is \(age)"
    ```
* traversals
  * only non-branching?
* knowable ranges?
* 

There are also functions which one would expect to be included but aren't.
The reason is because often these have implicit costs.
Here are some examples:
* Arithmetic
  * There are implicit costs and failure conditions like:
    * overflow/underflow
    * divide by zero
    * undefined behavior when mixing different numeric types
* Unions
  * Such as:
    * Wrappers
    * Results
    * Optional
    * Exception Values
  * Any type of union implicitly has a memory cost.
    This is because either the union must use indirection or inconsistent memory access.
* 

In cases that a conflict is detected a comptime error will be emitted.
For example, consider computing the total of a `Array`:
```
arr       = [100, 100, 100]
//arr       = [rand(0, 255), rand(0, 255), rand(0, 255)]
total :U8 = arr/0 + arr/1 + arr/2
```
The compiler will emit an error b/c it can be determined that total will overflow.



Unknowable
* non-deterministic
* mutation
* potentially invalid
  * undefined behaviour
  * overflow/underflow
  * divide-by-zero
  * infinities
  * error/exception free
* unclear dev intent
  * eg overflow is implicit in `U8` but should be explicit in behavior?
  * usage in each case but also related to data itself. It really depends on use-case?

Types of knowable functions:
* internal merge?
  * fold?
  * or? doesn't this have a cost b/c union has a memory cost?
  * known safe ops?
* range?
  * values which are known to not cause issues w/ specific ops?
* output known to be in some type
  * eg `"my age is \(age)"` must be `Int -> String`
* total?
  * every input must be able to map to an output
  * all output of the same type?
* edge-cases have been tested and modeled as output?
  * eg `Optional`? like `3 / x` becomes `Optional(Float)` b/c `x` could be `0`?
  * could be all types of errors, each need to be modeled in some way!
  * could also be modeled as outputs to multiple domains which could both be written to?
  * Handling things like this have a cost thought? or can be erased?
* comptime known
  * literals
  * comptime/const
* known bounds
  * this is related to comptime known
  * eg where you know the bounds will not be exceeded
    ```
    x = 4
    y = 6

    // never beyond U8 max and knowable
    z :U8 = sum(x, y)
    ```
* look into solvers (SMT)
* enumeration-functions but not their bodies
  * eg map is fine as long as body of map uses function which will always succeed
  * but there's also a cost in just enumeration (eg ADT or Tagged Union?)
    * even representing mixed memory is an issue? dev must be explicit?
* function must always succeed
  * or at least map them to an error value and union them?
* 

They must be:
* total
* associate?
* commutative?
* deterministic?


## Lambda
------------------------------------------------------------------------------------------------------------

Notes:
* usage
  * pattern matching
    * how names are passed to a handler where the handler is a type of lambda?
  * 
* ideally the function and lambda syntax are nearly the same?
  * that way the meta pattern is `fn-name = lambda-syntax`?
    * with an optional name or body? how to differentiate? just the last expression is always the body and if middle then it's function output?
      * like input, optional output, must include body
    * if `Fn` then read as `(in -> out, body)` but if not then is lambda of `(in -> body)`?
* syntax
  ```
  // Function delimiter?
  (x, y) -> (x + x)

  // use `=>` (fat arrow)
  (x, y) => (x + y) // lambda?
  (x, y) =>         // lambda?
    x + y

  // use `#fn`
  #fn(x, y) -> (x + y)
  #fn(x, y) ->
    (x + y)

  #fn(x, y) | x + y
  #fn(x, y)
    (x + y)

  // use `#do`
  #do(x, y) -> (x + y)
  #do(x, y) ->
    (x + y)

  #do(x, y) | x + y
  #do(x, y) |
    (x + y)

  #f(x -> x + 1)

  x :Int = 0, y :Int = 1 -> abs(x - y)
  (x :Int = 0, y :Int = 1 -> abs(x - y))
  #f(x :Int = 0, y :Int = 1 -> abs(x - y))
  #fn(x :Int = 0, y :Int = 1 -> abs(x - y))

  #f(x, (x + 1))
  ```


## Pattern
------------------------------------------------------------------------------------------------------------

### Match

Rust
```
// Matching range
let x = 5;
let y = 6;

match x {
    1..=5 => println!("one through five"),
    6     => println!("six"),
    y = 7 => println!("y is " y),
    7 | 8 => println!("seven or eight"),
    _     => println!("something else")
}

// Destructuring
struct Point {
    x: i32,
    y: i32,
}

let p              = Point { x: 0, y: 7 };
let Point { x, y } = p;
assert_eq!(0, x);
assert_eq!(7, y);

// Matching Struct
let p = Point { x: 0, y: 7 };

match p {
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y } => println!("On neither axis: ({}, {})", x, y),
}

// Destructuring Enums
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

match msg {
    Message::ChangeColor(Color::Rgb(r, g, b)) => println!(
        "Change the color to red {}, green {}, and blue {}",
        r, g, b
    ),
    Message::ChangeColor(Color::Hsv(h, s, v)) => println!(
        "Change the color to hue {}, saturation {}, and value {}",
        h, s, v
    ),
    _ => (),
}

// Mixed destructure
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });

// Ignoring multiple elems
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}

// Ignorning multiple tuple elems
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, .., last) => {
        println!("Some numbers: {}, {}", first, last);
    }
}

// Guarded Match
let x = Some(5);
let y = 10;

match x {
    Some(50)          => println!("Got 50"),
    Some(n) if n == y => println!("Matched, n = {}", n),
    _                 => println!("Default case, x = {:?}", x),
}

println!("at the end: x = {:?}, y = {}", x, y);

// Conjoined condition/guard
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _              => println!("no"),
}

// At Operator (@)
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {}", id),
}

```

Haskell
```
addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)
addVectors (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)

head :: [a] -> a
head []    = error "Can't call head on an empty list, dummy!"
head (x:_) = x

tell :: (Show a) => [a] -> String  
tell []       = "The list is empty"
tell (x:[])   = "The list has one element: " ++ show x
tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y
tell (x:y:_)  = "This list is long. The first two elements are: " ++ show x ++ " and " ++ show y

capital :: String -> String
capital ""         = "Empty string, whoops!"
capital all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]

max :: (Ord a) => a -> a -> a
max a b
  | a > b     = a
  | otherwise = b

// Cases
case expression of pattern -> result  
                   pattern -> result  
                   pattern -> result  
                   ...  

describeList :: [a] -> String  
describeList xs = "The list is " ++ case xs of [] -> "empty."  
                                               [x] -> "a singleton list."   
                                               xs -> "a longer list."  
```

OCaml
```
type color = Red | Black
type 'a tree = Empty | Tree of color * 'a tree * 'a * 'a tree

let rebalance t = match t with
    | Tree (Black, Tree (Red, Tree (Red, a, x, b), y, c), z, d)
    | Tree (Black, Tree (Red, a, x, Tree (Red, b, y, c)), z, d)                                  
    | Tree (Black, a, x, Tree (Red, Tree (Red, b, y, c), z, d))
    | Tree (Black, a, x, Tree (Red, b, y, Tree (Red, c, z, d)))
        ->  Tree (Red, Tree (Black, a, x, b), y, Tree (Black, c, z, d))
    | _ -> t (* the 'catch-all' case if no previous pattern matches *)
```

Tozen Prototyping
```
Color = Enum(Red = _, Black = _)
Tree  = Enum
  Tree
    Empty
    Color, #a Tree, #a, #a Tree
```

```
// Types of spec:
// - implicit arg? (a la Idris)
// - capture (a la Idris)
// - explicit arg (a la Zig)
// - hole (a la Idris)
// - expression (do, #do)
//   - result is signature
//   - this is effectively codegen?
Tree = Enum
  Leaf
  Node = #do
    a :Type

    (Tree a, Tree a)
```

### Enum/(Generics/Polymorphism)

```
// OCaml
type 'a tree =
    | Leaf
    | Node of 'a tree * 'a * 'a tree;;

let t = Node (Node (Leaf, 1, Leaf), 2, Node (Node (Leaf, 3, Leaf), 4, Leaf));;

// Tozen
Tree = #do
  T :TypeCapture

  Enum
    Leaf = _
    Node = (Tree(T), T, Tree(T))

t = Node(Node(Leaf, 1, Leaf), 2, Node(Node(Leaf, 3, Leaf), 4, Leaf)) // all ()
t = Node Node(Leaf, 1, Leaf), 2, Node Node(Leaf, 3, Leaf), 4, Leaf   // drop outermost ()
t = Node Node Leaf, 1, Leaf; 2, Node Node Leaf, 3, Leaf; 4, Leaf     // drop all ()
```

### Enum/Memory

For the targets which manage memory, the layout of `Enum`s don't match the order of fields.

Instead the compiler attempts to create an optimal layout.

For those languages where memory layout needs to be explicit the developer can use modifiers like:
```
StopLight #(packed = False, = Enum
  Green  = 0
  Yellow = 1
  Red    = 2
```

### GADT

```
// Haskell
data Expr a where
    EBool  :: Bool     -> Expr Bool
    EInt   :: Int      -> Expr Int
    EEqual :: Expr Int -> Expr Int  -> Expr Bool

eval :: Expr a -> a

eval e = case e of
    EBool a    -> a
    EInt a     -> a
    EEqual a b -> (eval a) == (eval b)

expr1 = EEqual (EInt 2) (EInt 3)        -- the type of ex

// TODO: Tozen
```

## Core Stuctures
------------------------------------------------------------------------------------------------------------

OCaml
```
primitives: int, float, string, bool

// List
// - ordered, homogeneous
// - arbitrary size 
[[1; 2]; [3; 4]; [5; 6]];; // int list list

// Tuple
// - ordered, heterogeneous
// - sized
(true, 0.0, 0.45, 0.73, "french blue");; // bool * float * float * float * string

// Record
// - like labeled tuples
// - must contain all fields on instantiation
type colour = {
  websafe : bool;
  r : float;
  g : float;
  b : float;
  name : string;
};;

let b = {
  websafe = true;
  r = 0.0;
  g = 0.45;
  b = 0.73;
  name = "french blue"
};;

// Mutable Records
type person = {
  first_name  : string;
  surname     : string;
  mutable age : int
};;

let birthday p = p.age <- p.age + 1;;

// Mutable fixed-length array
let arr = [|1; 2; 3|];; // int array
arr.(0);;               // int = 1
arr.(0) <- 0;;          // unit = ()
arr;;                   // int array = [|0; 2; 3|]

// Custom Type
// - constructors w/ data
// - can be recursively defined
type colour =
    | Red
    | Green
    | Blue
    | Yellow
    | RGB of float * float * float
    | Mix of float * colour * colour;;

Mix (0.5, Red, Mix (0.5, Blue, Green));;

// Trees
type 'a tree =
    | Leaf
    | Node of 'a tree * 'a * 'a tree;;

let t = Node (Node (Leaf, 1, Leaf), 2, Node (Node (Leaf, 3, Leaf), 4, Leaf));;

// Expression
type expr =
    | Plus   of expr * expr  (* a + b *)
    | Minus  of expr * expr  (* a - b *)
    | Times  of expr * expr  (* a * b *)
    | Divide of expr * expr  (* a / b *)
    | Var    of string       (* "x", "y", etc. *);;

// n * (x + y)
Times (Var "n", Plus (Var "x", Var "y"));;

// 

```

## Union
------------------------------------------------------------------------------------------------------------

Q:
* Should a ADT (or GADT) replace both Enum and Union?
* There an efficiency case for Union? or just for c compatibility?
* All similar ideas:
  * Keyword
    * Names w/o values
  * Enum
    * Names potentially w/o values
      * needs to have an integer value?
    * or like a Range w/o names?
    * enums are also used for things like bit flags but we may be able to use other constructs?
  * ADT
    * Name w/ diff fields over a union
  * Tagged Union
    * Name w/ diff fields over a union
  * Range
    * ranges are over values but not over names?
  * Static Records
* Zig basics
  * Primitives
  * Constructs
    * Array
    * Vector
    * Pointer
    * Slice
    * Struct
    * Enum
    * Union
    * Optional
    * Error & ErrorSet
  * Behavior
