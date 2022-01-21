Developers decide the level of complexity of the syntax by opting into dialects that suite their needs.
Currently these are the features one may include:
* base: Required for any implementation as it contains the basics of the syntax
* types and literals: Adds type-like constraints including types, interfaces, traits, etc. and some sugar to represent the most common data representations
* schema: Adds schema descriptions
* DSL: Adds syntax and sugar to create DSLs more easily often with deterministic, compiletime execution
* exec: Adds remaining syntax and sugar to create a full language

This document only describes the ***Tozen syntax*** itself.
It doesn't tell you about the ***semantics*** (meaning) of the syntax.
The semantics are added once the syntax is used w/n a specific context.
For example, a sequence within the context of a HTML document are HTML nodes.
Clarifying the semantics and other language uses are part of the core language and not described here as they're beyond the scope of the syntax.
However, to help, there are some examples with psuedo-code at the end.

For example, to emulate JSON you would need to incorporate ***base and literal*** while for a different approach to emulate OGDL you must incoporate ***base, literals, and schema***.

[Basic Types, Literals, and Schema](./basic_types) How the most common types, literals, and schemas are defined in Tozen syntax.

[Limited Compile Time](./limited_comptime) How a limited compile time execution is modeled in Tozen which allows you to significantly reduce duplicated code and data.

[Compile Time, Advanced Types, and Metadata](./advanced_types) How compile time interacts with advanced types and metadata 

[Specification](./spec) How to create a better way to specify your code, data, and execution in a way somewhat different than most languages.

[DSL](./dsl) How to model DSLs.

[Executable](./exec) The remaining syntax to model a full language.

[Learn by Example](./examples) Read to learn the Tozen syntax standard using examples, counter examples, and how other languages compare to do the same tasks.

[Compare to Others](./compare) Compare Tozen syntax solutions to other language implementations.

[Issues, Edge Cases, and Conflicts](./issues) Outstanding issues with the syntax standards and examples of results which may be unexpected.

-------------------------------------------------------

Levels
* New Levels:
  * Based on exec complexity
    * No execution
    * No runtime
    * Finite runtime
    * Full Language
    * Meta Language
      * emits any previous level
  * Based on output comparable to other languages
    * Serialization-like
    * Dhall-like
    * Datalog-like (finite)
    * Full language
    * Meta Language
      * emits any previous level
  * 
    * Serialization-like
    * Compiler Erased (free abstraction)
    * Full language
    * Meta Language
      * emits any previous level
    * other?
      * zero-cost abstractions?
      * finite?
      * patterns?
      * user-defined vs system-defined?
  * What you wish to target, emulate, or recreate
  * Which guarantees can be provided
  * Which costs
  * Syntax and semantics necessary to be added/modeled
  * 
  * 
* Notes
  * any type of comptime execution nearly always same syntax/semantic complexity of runtime so they should be grouped into the same level?
    * this likely happens often where diff aspects gain/lose complexity at different levels (so just group together)
  * don't introduce simpler syntax which will conflict with later syntax levels
  * builtin concepts vs user definable?
    * related to what can be known and intrinsic?
  * 
