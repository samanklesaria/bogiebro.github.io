---
layout: post
---

When programming, I like it when my unit tests run whenever I save a file. In the spirit of Bret Victor's [Inventing on Principle](https://vimeo.com/36579366), this gives me immediate feedback if whatever I just wrote broke something. Unfortunately, Julia doesn't naturally support this workflow. Testing is normally done interactively, by entering the `test` command into a Julia repl, so it's not something that's easy to plug into [watchexec](https://github.com/watchexec/watchexec), like I usually do. On top of this problem, the `test` command will reload your package from scratch every time it's run. In Julia, loading a package take ages, making immediate feedback impossible.



To escape this unpleasantness, I made the package [ReTester](https://github.com/bogiebro/ReTester), to be used with [Revise](https://timholy.github.io/Revise.jl/stable/). The proposed workflow goes like this:

1. Make a `test/runtests.jl` file as usual, but don't put your tests there directly. Instead, `includet` a separate tests file, allowing Revise to watch for changes. If you're writing the package `MyPackage`,  `runtests.jl` should look something like this:

   ```julia
   using Revise
   using ReTester
   using MyPackage
   includet("tests.jl")
   entr(run_tests(test), ["src", "test"]; pause=0.2, all=true)
   ```

   and `tests.jl` should contain a `test` function to run all the tests. 

2. Add `Revise` and `ReTester` to your test dependencies.

3. Run the `test` command at a Julia repl as usual. Your tests will get re-run when any of your files change, using the fast environment patching from Revise instead of reloading your module from scratch.

Happy testing! 