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
In principle, you can always solve this problem in the following way;
```julia
using MPI

abstract type Rank{N} end

function func(rank)
    if rank <: Rank{0}
        # job for root
    elseif rank <: Rank{1}
        # job for rank 1
    elseif rank <: Rank{2}
        # job for rank 2
    elseif rank <: Rank{3}
        # job for rank 3
    end
end

MPI.Init()

comm = MPI.COMM_WORLD
rank = Rank{MPI.Comm_rank(comm)}
func(rank)

MPI.Finalize()
```
Or it is better to write in the following way as suggested by [the official document](https://docs.julialang.org/en/v1/manual/performance-tips/#Break-functions-into-multiple-definitions):
```julia
using MPI

abstract type Rank{N} end

function func(rank::Type{<:Rank{0}})
    # job for root
end

function func(rank::Type{<:Rank{1}})
    # job for rank 1
end

function func(rank::Type{<:Rank{2}})
    # job for rank 2
end

function func(rank::Type{<:Rank{3}})
    # job for rank 3
end

MPI.Init()

comm = MPI.COMM_WORLD
rank = Rank{MPI.Comm_rank(comm)}
func(rank)

MPI.Finalize()
```

This is how to work around. However, to me it looks like overengineering.

## Load balancing
