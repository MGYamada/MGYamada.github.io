+++
title = "MPI in Julia"
hascode = true
date = Date(2021, 9, 11)
+++
@def tags = ["syntax", "code"]

# MPI in Julia

\toc

## MPI.jl

MPI is a message passing API for the distributed memory parallelization.
MPI has many implimentations, such as Intel MPI, Open MPI, etc. One of the
advantage of MPI.jl is that it supports various implimantations of MPI flexibly,
and it is easy to use MPI from Julia by preparing functions with almost the same
name as C functions, such as `MPI.Send`. Additionally MPI.jl offers a simplified
version `MPI.send` which internally serialize the Julia object.

MPI.jl also supports the hybrid parallelization. You can request the thread-safe
level during the initialization. Essentially, you can write a code with both MPI
parallelization and Julia parallelization. MPI.jl also supports CUDA-aware MPI.

## Basic functions

## "Lower-case" functions

## Type stability

## Type-stable way to use conditional branches for `rank`

Sometimes the conditional branch for `rank` causes unavoidable type instability.
In principle, you can always solve this problem in the following way:
```julia
using MPI

function func(::Val{rank}) where rank
    if rank == 0
        # job for root
    elseif rank == 1
        # job for rank 1
    elseif rank == 2
        # job for rank 2
    elseif rank == 3
        # job for rank 3
    end
end

MPI.Init()

comm = MPI.COMM_WORLD
rank = MPI.Comm_rank(comm)
func(Val(rank))

MPI.Finalize()
```
Or it is better to write in the following way as suggested by [the official document](https://docs.julialang.org/en/v1/manual/performance-tips/#Break-functions-into-multiple-definitions):
```julia
using MPI

function func(::Val{0})
    # job for root
end

function func(::Val{1})
    # job for rank 1
end

function func(::Val{2})
    # job for rank 2
end

function func(::Val{3})
    # job for rank 3
end

MPI.Init()

comm = MPI.COMM_WORLD
rank = MPI.Comm_rank(comm)
func(Val(rank))

MPI.Finalize()
```

This is how to work around. In either case, the JIT compiler will compile a necessary part of your code.

## Load balancing
