---
layout: default
---

Tozen's goal is to be a full-stack, optimal, and meta-programmable language.
A language that can be used from systems programming to front-end styling.

It achieves this trick by combining a few techniques:
* stratification
  * chose minimal amount of features in order for compiler providing more guarantees
* data-oriented syntax (ripe for external tools, analysis, and synthesis)
* comptime, MSP (both? others? fexpr?)
* contexts, dialecting, and reinterpreting syntax
* others?
  * rulesets
    * some provable and some not

Tozen is stratified from a simple data-description language to a general purpose language:
* Base
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
* Erasable (or at least nearly? or fully?)
  * features:
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
* Finite
  * features:
  * potentially:
    * deterministic (time, memory, execution)
    * constrained/finite dependent types
    * turing-incompleteness
  * comparable to:
* Comptime Only (no runtime cost, memory, or execution)
  * features:
  * potentially:
  * comparable to:
* General Purpose
  * features:
  * potentially:
  * comparable to:

Tozen documentation is broken into a few parts.

## Syntax
[Base](./base) The essentials of Tozen syntax which is necessary for every implementation. Is meant for program synthesis not developers.

[Base Sugar](./base_sugar) Ergonomic version of [Base](./base) meant for developers and not program synthesis.

