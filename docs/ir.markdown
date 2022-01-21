---
layout: default
title: IR
description: Universal, consistent, and explicit representation of Tozen Language
permalink: ir
---

The most explicit level of Tozen is its internal intermediate representation (IR).
Although developers are free to write Tozen using IR, we provide the higher-level, ergonomic, sugar levels which ultimately translate into this IR.

Many higher level syntaxes are more expressive but ultimately reduce to this core syntax.
Keeping the set of core elements simple makes it easier for tools to be developed for it.
It is also rare that a developer will use it directly as the higher level syntaxes are more ergonomic.

This is also how we interface with many other languages.


## Basic Types and Literals
------------------------------------------------------------------------------------------------------------

Tozen reserves some common types to assure they have a universal meaning.

This doesn't mean they have the same cost or representation on each build target.
For example, `I8` is implemented differently in `JavaScript` compared to the `JVM`.
When types are available on a target, they are guaranteed to function the same even if have different costs.

### Basic Types

Basic types are directly represented by the environment they run on.
Higher level types are often built using these.
Often the target is LLVM but can also target many others like JavaScript, Java, the HTML DOM, etc.

* Numeric
  * Integral
    * `Int` is the target environment default signed, integer (32 or 64 bit)
    * `UInt` is the target environment default unsigned, integer (32 or 64 bit)
    * `Nat` are positive `Int` (not including zero)
    * `Whole` are positive `Int` (including zero)
    * `I8, I16, I32, I64, I128` are sized, signed integers
    * `U8, U16, U32, U64, U128` are sized, unsigned integers
  * Floating Point (IEEE 754)
    * `Float` is the target environment default floating point (32 or 64 bit)
    * `F8, F16, F32, F64` are sized, signed floating point
  * Decimal or Fixed Point
    * Exact decimal until overflow
    * `Fixed` is the target environment default fixed point (32 or 64 bit)
    * `Fx8, Fx16, Fx32, Fx64` are sized, signed fixed point
    * `UFx8, UFx16, UFx32, UFx64` are sized, unsigned fixed point
  * Fraction
    * Exact fraction until overflow
    * `Frac` is the target environment default fraction (32 or 64 bit)
    * `Frac8, Frac16, Frac32, Frac64` are sized, signed fixed point
    * `UFrac8, UFrac16, UFrac32, UFrac64` are sized, unsigned fixed point
* String
  * `Verbatim` is for exact Strings
  * `StringLiteral` is for escapable Strings
* `Bool`
  * Only possible values are `True` and `False` literal
* `None`
  * Only possible value is `None` literal


## Literal Interpretation
------------------------------------------------------------------------------------------------------------

When literals are parsed they are interpreted by the compiler.
This process fills in relator fields to describe the interpretation.

Here are simple examples for each of the builtin literals:
* `Integral-Literal`
  * eg interprets `x = 125` as:
    ```
    x
      :type       = Integral-Literal
      :is-signed  = False
      :num-digits = 3
    ```
* `Decimal-Literal`
  * eg interprets `x = -12.6852` as:
    ```
    x
      :type               = Decimal-Literal
      :is-signed          = True
      :num-whole-digits   = 3
      :num-decimal-digits = 3
    ```
* `Fraction-Literal`
  * eg interprets `x = 22./7` as:
    ```
    x
      :type        = Fraction-Literal
      :is-signed   = False
      :numerator   = 3
      :denominator = 3
      :is-rational = Unknown // True, False, and Unknown? can't always be deteremined?
    ```
* `Percent-Literal`
  * eg interprets `x = 12.34%` as:
    ```
    x
      :type               = Percent-Literal
      :is-signed          = False
      :num-whole-digits   = 3
      :num-decimal-digits = 3
    ```
* `Binary-Literal`
  * eg interprets `x = 0b1010` as:
    ```
    x
      :type       = Binary-Literal
      :is-signed  = False
      :num-digits = 4
    ```
* `Octal-Literal`
  * eg interprets `x = 0o744` as:
    ```
    x
      :type       = Octal-Literal
      :is-signed  = False
      :num-digits = 3
    ```
* `Hex-Literal`
  * eg interprets `x = 0xF00D` as:
    ```
    x
      :type       = Hex-Literal
      :is-signed  = False
      :num-digits = 4
    ```
* `Exponent-Literal`
  * eg interprets `x = -4e-2.5` as:
    ```
    x
      :type      = Exponent-Literal
      :is-signed = True
      :significand
        :type       = Integral-Literal
        :is-signed  = True
        :num-digits = 1
      :exponent
        :type               = Decimal-Literal
        :is-signed          = True
        :num-whole-digits   = 1
        :num-decimal-digits = 1
    ```
