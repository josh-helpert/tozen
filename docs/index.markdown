---
layout: default
---

Tozen syntax's goal is to unify homoiconic and data-oriented syntax structure as tersely as possible and ultimately create a general purpose syntax. The Tozen core language uses the Tozen syntax as it's choice of syntax much like many LISP implementations use S-expressions as their syntax.

Tozen syntax is a general purpose syntax which competes with a variety of syntax use-cases:
* homoiconic syntaxes like s-expressions, sweet-expressions, o-expressions, Rebol syntax
* serialization and data-modeling
  * syntax and literal types like JSON, YAML, SDLang, OGDL
  * value-only syntaxes like StrictYAML
  * schema like InternetObject, OGDL
  * custom DSL like Pug, SASS, Markdown

Implementors decide the level of complexity of the syntax by opting into dialects that suite their needs. Currently these are the features one may include:
* base: Required for any implementation as it contains the basics of the syntax
* types and literals: Adds type-like constraints including types, interfaces, traits, etc. and some sugar to represent the most common data representations
* schema: Adds schema descriptions
* DSL: Adds syntax and sugar to create DSLs more easily often with deterministic, compiletime execution
* exec: Adds remaining syntax and sugar to create a full language

For example, to emulate JSON you would need to incorporate ***base and literal*** while for a different approach to emulate OGDL you must incoporate ***base, literals, and schema***.

This document only describes the ***Tozen syntax*** itself. It doesn't tell you about the ***semantics*** (meaning) of the syntax. The semantics are added once the syntax is used w/n a specific context. For example, a sequence within the context of a HTML document are HTML nodes. Clarifying the semantics and other language uses are part of the core language and not described here as they're beyond the scope of the syntax. However, to help, there are some examples with psuedo-code at the end.


Tozen syntax documentation is broken into a few parts.

## Syntax
[Base Syntax](./base) Overview of the base of Tozen syntax which is necessary for every implementation.

[Basic Types, Literals, and Schema](./basic_types) How the most common types, literals, and schemas are defined in Tozen syntax.

[Limited Compile Time](./limited_comptime) How a limited compile time execution is modeled in Tozen which allows you to significantly reduce duplicated code and data.

[Compile Time, Advanced Types, and Metadata](./advanced_types) How compile time interacts with advanced types and metadata 

[Specification](./spec) How to create a better way to specify your code, data, and execution in a way somewhat different than most languages.

[DSL](./dsl) How to model DSLs.

[Executable](./exec) The remaining syntax to model a full language.


[Learn by Example](./examples) Read to learn the Tozen syntax standard using examples, counter examples, and how other languages compare to do the same tasks.

[Compare to Others](./compare) Compare Tozen syntax solutions to other language implementations.

[Issues, Edge Cases, and Conflicts](./issues) Outstanding issues with the syntax standards and examples of results which may be unexpected.
