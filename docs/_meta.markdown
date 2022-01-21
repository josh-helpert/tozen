
TODO:
* Suffixing/priming like `` x` = `1 + 2` `` to represent levels of code abstraction?
  * then same rules as priming like `x'5` to have many levels?
* 
* 

Notes:
* MSP/meta/comptime gives developer explicit control instead of hoping compiler, intrinsics, etc.
*
*

Q:
* Need to decide which type(s) of meta to use
  * benefits of having many is we can use the simplest we need
  * downsides of having many is they can conflict
* Expose the compiler as a lib/series of services to the developer?
  * then they can use in specific ways
* Require that something is statically determinable by comptime
  * this just mean we have a `static` keyword to add the constraint?
  * eg want an integer type which is statically analyzable so can use freely
  * monomorphic restriction?
  * substitutable restriction?
*
*
