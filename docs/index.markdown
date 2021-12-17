---
layout: default
---

Tozen's goal is to be a full-stack, meta-programmable, and optimal language.
A language that can be used from front-end to systems programming.

It achieves this trick by combining a few techniques:
* stratification
  * the language is broken into different levels
  * each level grows in functionality while reducing compiler guarantees
* data-oriented syntax
  * is it easier to create external tools, analysis, and synthesis
  * similar to how S-expressions are easy to reason on
* zero-cost abstractions with metaprogramming
  * comptime, MSP (both? others? fexpr?)
* others?
  * contexts, dialecting, and reinterpreting syntax
  * rulesets
    * some provable and some not
  * integration with multiple targets

Each stratified level is broken into two parts:
* IR
  * a minimal, consistent, and precise language
  * useful for analysis and generation by other programs
  * only uses lower levels of IR to keep simple
* sugar
  * ergonomic, productive, and terse language
  * useful for developers
  * uses lower levels of sugar
  * translates to IR

As levels grow in complexity they lose compiler guarantees.
It is often best to use the simplest level possible.
This allows the compiler to do more optimizations and verification on your behalf.

Tozen is divided by level and part:
* Level 0 - Core [IR](./core) [Sugar](./core_sugar)
  * description
    * essentials of Tozen syntax which is necessary for every implementation
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
    * execution?
    * deterministic (time, memory, execution)
    * constrained/finite dependent types
    * turing-incompleteness
    * operators
    * 
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
