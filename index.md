@def title = "Julia in High-Performance Computing"
@def tags = ["syntax", "code"]

# Julia in High-Performance Computing

This is a website on how to use Julia in high-performance computing.
I will give examples which use Julia in supercomputers, GPGPUs, etc.

Julia is a modern multi-paradigm language with various extensions
for OOP and functional programming. This allows us to do a modern
style programming even in the world of high-performance computing (HPC),
where usually C/C++ and Fortran have been used for a long time.
Especially, Julia's implementation of the multiple dispatch is so
powerful that you can write, for example, a code which works both
in CPU and GPU at the same time without writing an exclusive code.

Here I focus on MPI.jl and CUDA.jl libraries which allow us to
do most of the HPC functionalities.