+++
title = "CUDA in Julia"
hascode = true
date = Date(2021, 9, 11)
+++
@def tags = ["syntax", "code"]

# CUDA in Julia

\toc

## CUDA.jl

## How to rewrite a CPU code to a GPU code

* Add `CUDA.` for initialization functions.
    * `zeros(m, n)` -> `CUDA.zeros(Float64, m, n)`
* Sometimes call a function `CUDA.synchronize()`.
* Call kernel functions with `@cuda` macro.
* Replace BLAS with CUBLAS.
* Replace LAPACK with cuSOLVER or MAGMA. (I recommend MAGMA.)
* Please be aware where your data are, on CPU, or on GPU.

## How to write a kernel function

## Type-unstable way to make CPU and GPU codes coexist

## Type-stable way to make CPU and GPU codes coexist

Unfortunately, the above way of switching CPU and GPU by the `Bool` value `gpu` causes type instability.
There are many ways to avoid this problem. Here's one simple way to make it type-stable.

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
This automatically enables us to give CPU or GPU information to the compiler. All the `if gpu` syntax are
inferred from the type of inputs, so it is enough to make everything type-stable.

One advantage of using abstract types is that you can add your own "sub-engines" to `CPUEngine` or `GPUEngine`.
For example, you can add:
```julia
abstract type MultiGPUEngine <: GPUEngine end
```
to write a multi-GPU code.
