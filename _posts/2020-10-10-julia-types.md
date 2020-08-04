{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Static Typing in Julia"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "We don't need to run this function to know that we'll get a MethodError. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "bad_f1 (generic function with 1 method)"
      ]
     },
     "execution_count": 1,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "function bad_f1()\n",
    "    [] + 5\n",
    "end"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Fortunately, Julia can easily figure out that this code is broken without running it."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "1-element Array{Any,1}:\n",
       " CodeInfo(\n",
       "   \u001b[33m @ In[1]:2 within `bad_f1'\u001b[39m\n",
       "   \u001b[33m┌ @ array.jl:129 within `vect'\u001b[39m\n",
       "   \u001b[33m│┌ @ boot.jl:425 within `Array' @ boot.jl:406\u001b[39m\n",
       "\u001b[90m1 ─\u001b[39m\u001b[33m││\u001b[39m %1 = $(Expr(:foreigncall, :(:jl_alloc_array_1d), Array{Any,1}, svec(Any, Int64), 0, :(:ccall), Array{Any,1}, 0, 0))\u001b[36m::Array{Any,1}\u001b[39m\n",
       "\u001b[90m│ \u001b[39m \u001b[33m└└\u001b[39m\n",
       "\u001b[90m│  \u001b[39m      (%1 + 5)\u001b[90m::Union{}\u001b[39m\n",
       "\u001b[90m└──\u001b[39m      $(Expr(:unreachable))\u001b[90m::Union{}\u001b[39m\n",
       ") => Union{}"
      ]
     },
     "execution_count": 2,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "code = code_typed(bad_f1; debuginfo=:source)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "This also works for manual return type annotations, which is just syntactic sugar for an explicit coersion at the end of the function."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "bad_f2 (generic function with 1 method)"
      ]
     },
     "execution_count": 3,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "function bad_f2()::Int\n",
    "    []\n",
    "end"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "1-element Array{Any,1}:\n",
       " CodeInfo(\n",
       "\u001b[90m1 ─\u001b[39m %1 = Main.Int\u001b[36m::Core.Compiler.Const(Int64, false)\u001b[39m\n",
       "\u001b[90m│  \u001b[39m %2 = $(Expr(:foreigncall, :(:jl_alloc_array_1d), Array{Any,1}, svec(Any, Int64), 0, :(:ccall), Array{Any,1}, 0, 0))\u001b[36m::Array{Any,1}\u001b[39m\n",
       "\u001b[90m│  \u001b[39m      Base.convert(%1, %2)\u001b[90m::Union{}\u001b[39m\n",
       "\u001b[90m└──\u001b[39m      $(Expr(:unreachable))\u001b[90m::Union{}\u001b[39m\n",
       ") => Union{}"
      ]
     },
     "execution_count": 4,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "code_typed(bad_f2)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "To automate these checks, we just look through our module at all the methods, and complain about the unreachable part when they produce `Union{}`."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "check_types (generic function with 1 method)"
      ]
     },
     "execution_count": 5,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "unreachable_stmt = :($(Expr(:unreachable)))\n",
    "function check_types()\n",
    "    for funcname in names(Main)\n",
    "        for (code, ty) in code_typed(eval(funcname); debuginfo=:source)\n",
    "            if ty == Union{}\n",
    "                for (i, stmt) in enumerate(code.code)\n",
    "                    if stmt == unreachable_stmt\n",
    "                        loc = code.linetable[code.codelocs[i]]\n",
    "                        println(stderr, \"No appropriate method in \", String(funcname), \" (\", loc.file, \":\", loc.line, \")\")\n",
    "                        break\n",
    "                    end\n",
    "                end\n",
    "            end\n",
    "        end\n",
    "    end\n",
    "end"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "No appropriate method in bad_f1 (In[1]:2)\n",
      "No appropriate method in bad_f2 (In[3]:2)\n"
     ]
    }
   ],
   "source": [
    "check_types()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "We can hook this into Revise's reloads, allowing the kind of static type checking that users of Haskell's `ghcid` and similar tools are used to. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "using Revise\n",
    "Revise.entr(check_types, []; all=true)"
   ]
  }
 ],
 "metadata": {
  "@webio": {
   "lastCommId": "24efac7dcc2c406e82e58fee5eee3484",
   "lastKernelId": "b6955904-07d1-4723-a7cf-7694e353796f"
  },
  "kernelspec": {
   "display_name": "Julia 1.5.2",
   "language": "julia",
   "name": "julia-1.5"
  },
  "language_info": {
   "file_extension": ".jl",
   "mimetype": "application/julia",
   "name": "julia",
   "version": "1.5.2"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
