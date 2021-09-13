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

## Load balancing
