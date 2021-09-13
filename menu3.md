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
