---
layout: default
---

Tozen's goal is to be a full-stack, meta-programmable, and optimal language.
A language that can be used from systems programming to front-end styling.

It achieves this trick by combining a few techniques:
* stratification
  * chose minimal amount of features in order for compiler providing more guarantees
* data-oriented syntax
  * is it easier to create external tools, analysis, and synthesis
  * similar to how S-expressions are easy to reason on
* zero-cost abstractions
  * comptime, MSP (both? others? fexpr?)
* contexts, dialecting, and reinterpreting syntax
  * 
* others?
  * rulesets
    * some provable and some not

Each level is broken into two parts:
* the IR level which is a minimial, consistent language useful for analysis and generation by programs
* the sugar level which is a ergonomic, productive, and terse language useful for developers

As levels grow in complexity they lose compiler guarantees.
It is often best to use the simplest level possible.
This allows the compiler to do more optimizations and verification on your behalf.

The levels are:
* Level 0 - Core
  * description
    * 
  * features:
    * literals
    * records
    * relators
    * line continuation
    * comments and docstrings
  * potentially:
  * comparable to:
    * homoiconic syntaxes like s-expressions, sweet-expressions, o-expressions, Rebol syntax
    * serialization and data-modeling
      * literals like JSON, YAML, SDLang, OGDL
      * value-only syntaxes like StrictYAML
* Level 1 - Compiler Erased
  * description
    * 
  * features:
    * relator select
    * variables
    * substitution
    * simple logic
    * FSM
  * potentially:
    * types
    * schema
    * refinement types
    * constrainted/finite dependent types
    * total functions
  * comparable to:
    * schema like InternetObject, OGDL
    * custom DSL like Pug, SASS, Markdown
* Level 2 - Finite
  * description
    * 
  * features:
    * 
  * potentially:
    * deterministic (time, memory, execution)
    * constrained/finite dependent types
    * turing-incompleteness
  * comparable to:
    * 
* Level 3 - Comptime Only (no runtime cost, memory, or execution)
  * description
    * 
  * features:
    * 
  * potentially:
    * 
  * comparable to:
    * 
* Level 4 - General Purpose
  * description
    * 
  * features:
    * 
  * potentially:
    * 
  * comparable to:
    * 

Tozen documentation is broken into a few parts.

## Syntax
[Base IR](./base) The essentials of Tozen syntax which is necessary for every implementation. Is meant for program synthesis not developers.

[Base Sugar](./base_sugar) Ergonomic version of [Base IR](./base) meant for developers and not program synthesis.
