---
layout: post
---

We don't need to run this function to know that we'll get a MethodError. 


```julia
function bad_f1()
    [] + 5
end
```


Fortunately, Julia can easily figure out that this code is broken without running it.


```julia
> code_typed(bad_f1; debuginfo=:source)
1-element Array{Any,1}:
 CodeInfo(
   [33m @ In[1]:2 within `bad_f1'[39m
   [33m┌ @ array.jl:129 within `vect'[39m
   [33m│┌ @ boot.jl:425 within `Array' @ boot.jl:406[39m
[90m1 ─[39m[33m││[39m %1 = $(Expr(:foreigncall, :(:jl_alloc_array_1d), Array{Any,1}, svec(Any, Int64), 0, :(:ccall), Array{Any,1}, 0, 0))[36m::Array{Any,1}[39m
[90m│ [39m [33m└└[39m
[90m│  [39m      (%1 + 5)[90m::Union{}[39m
[90m└──[39m      $(Expr(:unreachable))[90m::Union{}[39m
    ) => Union{}
```



This also works for manual return type annotations, which is just syntactic sugar for an explicit coersion at the end of the function.


```julia
function bad_f2()::Int
    []
end
```



```julia
> code_typed(bad_f2)
1-element Array{Any,1}:
 CodeInfo(
[90m1 ─[39m %1 = Main.Int[36m::Core.Compiler.Const(Int64, false)[39m
[90m│  [39m %2 = $(Expr(:foreigncall, :(:jl_alloc_array_1d), Array{Any,1}, svec(Any, Int64), 0, :(:ccall), Array{Any,1}, 0, 0))[36m::Array{Any,1}[39m
[90m│  [39m      Base.convert(%1, %2)[90m::Union{}[39m
[90m└──[39m      $(Expr(:unreachable))[90m::Union{}[39m
) => Union{}
```



To automate these checks, we just look through our module at all the methods, and complain about the unreachable part when they produce `Union{}`.


```julia
unreachable_stmt = :($(Expr(:unreachable)))
function check_types()
    for funcname in names(Main)
        for (code, ty) in code_typed(eval(funcname); debuginfo=:source)
            if ty == Union{}
                for (i, stmt) in enumerate(code.code)
                    if stmt == unreachable_stmt
                        loc = code.linetable[code.codelocs[i]]
                        println(stderr, "No appropriate method in ", String(funcname), " (", loc.file, ":", loc.line, ")")
                        break
                    end
                end
            end
        end
    end
end
```

```console
> check_types()
No appropriate method in bad_f1 (In[1]:2)
No appropriate method in bad_f2 (In[3]:2)
```


We can hook this into Revise's reloads, allowing the kind of static type checking that users of Haskell's `ghcid` and similar tools are used to. 


```julia
using Revise
Revise.entr(check_types, []; all=true)
```
