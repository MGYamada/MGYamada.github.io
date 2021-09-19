+++
title = "CUDA in Julia"
hascode = true
date = Date(2021, 9, 11)
+++
@def tags = ["syntax", "code"]

# CUDA in Julia

\toc

## CUDA.jl

CUDA.jl is the Julia version of the CUDA library, which offers a GPGPU plarform for NVIDIA GPU.
CUDA.jl is not just a wrapper for CUDA functions, but also has functionality to compile the kernel code.
CUDA.jl completely natively supports Julia, i.e. you can use a native Julia code to write a kernel function
like a script language. The JIT compilation completely works.

## How to rewrite a CPU code to a GPU code

* Add `CUDA.` for initialization functions.
    * `zeros(m, n)` -> `CUDA.zeros(Float64, m, n)`
    * `rand(m, n)` -> `CUDA.rand(Float64, m, n)`
* Sometimes call a function `CUDA.synchronize()`.
* Call kernel functions with `@cuda` macro.
* Replace BLAS with CUBLAS.
* Replace LAPACK with cuSOLVER or MAGMA. (I recommend [MAGMA](https://icl.cs.utk.edu/magma/).)
* Please be aware where your data are, on CPU, or on GPU.

These are the only things you have to do to make your CPU code work in GPU.
CUDA versions are included in CUDA.jl for most of the `Base`/`LinearAlgebra` functions.
Essentially, just by changing initialization functions, Julia's multiple dispatch automatically
changes CPU methods to GPU methods. Only you have to do is to be aware of types, `Array` or `CuArray`,
`Vector` or `CuVector`, `Matrix` or `CuMatrix`. This nice switch with the same functions
thanks to the multiple dispatch paradigm even allows us to write a code which works both in CPU
and in GPU at the same time.

## How to write a kernel function

## Type-unstable way to make CPU and GPU codes coexist

You can easily make CPU and GPU codes coexist in your own code.
Here's an example.
```julia
using LinearAlgebra
using CUDA

function power_iteration(m, gpu)
    if gpu
        A = CUDA.rand(m, m)
        b = CUDA.rand(m)
    else
        A = rand(Float32, m, m)
        b = rand(Float32, m)
    end

    for i in 1:100
        b .= A * b
        b ./= norm(b)
    end

    dot(b, A * b)
end
```

The code works in GPU when `gpu = true` and in CPU when `gpu = false`. The point is that you do not have
to write the main algorithm of the power iteration twice. It is enough to have an `if` syntax in the initialization.
However, this code is very type-unstable. You can check that by `@code_warntype power_iteration(100, true)` for example.

## Type-stable way to make CPU and GPU codes coexist 1

Unfortunately, the above way of switching CPU and GPU by the `Bool` value `gpu` causes type instability.
There are many ways to avoid this problem. Here's the simplest way to make it type-stable.

The simplest way is to use a `Val` type. The change is easy, just give a value `gpu` via the `Val(gpu)`.
```julia
using LinearAlgebra
using CUDA

function power_iteration(m, ::Val{gpu}) where gpu
    if gpu
        A = CUDA.rand(m, m)
        b = CUDA.rand(m)
    else
        A = rand(Float32, m, m)
        b = rand(Float32, m)
    end

    for i in 1:100
        b .= A * b
        b ./= norm(b)
    end

    dot(b, A * b)
end
```

Just call this function via `power_iteration(100, Val(true))`. This is the code with the smallest change.

## Type-stable way to make CPU and GPU codes coexist 2

Another solution is the following. It is more complicated but more sophisticated.
First, define the following abstract types:
```julia
abstract type Engine end
abstract type CPUEngine <: Engine end
abstract type GPUEngine <: Engine end
```
These abstract types seem meaningless, but they have an important role to give information to the compiler.
Then, you should give an argument `engine` instead of `gpu`. `engine` should only take `CPUEngine` or `GPUEngine`,
depending on which kernel you use.

Finally, it is enough to add the following one line at the beginning of every function:
```julia
gpu = engine <: GPUEngine
```
This automatically enables us to give CPU or GPU information to the compiler. Every `if gpu` syntax is
inferred from the type of inputs, so it is enough to make everything type-stable.

```julia
using LinearAlgebra
using CUDA

abstract type Engine end
abstract type CPUEngine <: Engine end
abstract type GPUEngine <: Engine end

function power_iteration(m, engine)
    gpu = engine <: GPUEngine

    if gpu
        A = CUDA.rand(m, m)
        b = CUDA.rand(m)
    else
        A = rand(Float32, m, m)
        b = rand(Float32, m)
    end

    for i in 1:100
        b .= A * b
        b ./= norm(b)
    end

    dot(b, A * b)
end
```

One advantage of using abstract types is that you can add your own "sub-engines" to `CPUEngine` or `GPUEngine`.
For example, you can add:
```julia
abstract type MultiGPUEngine <: GPUEngine end
```
to write a multi-GPU code.
