---
title: 'Astaroth: A scientific computing framework for accelerating stencil computations.'
tags:
  - C/C++
  - scientific computing
  - high performance computing
  - astrophysics

authors:
  - name: Touko Puro
    orcid: 0000-0000-0000-0000
    corresponding: true
    affiliation: 1 
  - name: Johannes Pekkilä
    orcid: 0000-0000-0000-0000
    affiliation: 1 
  - name: Miikka Väisälä
    orcid: 0000-0000-0000-0000
    affiliation: 2
  - name: Oskar Lappi 
    orcid: 0000-0000-0000-0000
    affiliation: 3
  - name: Matthias Rheinhardt
    orcid: 0000-0000-0000-0000
    affiliation: 1
  - name: Hsien Shang
    orcid: 0000-0000-0000-0000
    affiliation: 4
  - name: Maarit Korpi-Lagg 
    orcid: 0000-0000-0000-0000
    affiliation: 1

affiliations:
 - name:  Aalto University, Finland 
   index: 1
 - name:  University of Oulu, Finland 
   index: 2
 - name:  University of Helsinki, Finland 
   index: 3
 - name:  Academia Sinica Institute of Astronomy and Astrophysics, Taiwan
   index: 4
 
date: 20 April 2026
bibliography: paper.bib

---

# Summary

Stencil computations[^stencil_footnote] are one of the bedrocks of high performance scientific simulations, forming the core of many partial differential equation (PDE) and numerical linear algebra solvers. 
In recent years, GPUs have become the primary compute platform for high-performance computing, and it is difficult to run large simulations without them.
`Astaroth` is a GPU framework for stencil computations, that has been developed with scalable scientific computing in mind.

`Astaroth` provides its own domain specific language (DSL), in which researchers can express their required computations without having to focus on technical implementation details.
It can run efficiently both on CUDA- and HIP-based environments --- and even on hardware lacking GPUs, e.g. for testing purposes.
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing and has library integrations for performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
To further ease the development and acceleration of PDE solvers based on the finite-difference method, `Astaroth` comes together with its own PDE solver and a standard library with support for a variety of PDE-solver functionality.
`Astaroth` has primarily been used for turbulent astrophysical plasma simulations.

# Statement of need

Much of the software used for scientific computing is written for CPUs, and has to be ported to GPUs to run larger problems.
`Astaroth` has been developed to solve this problem for the subset of scientific software that can be expressed as stencil computations.
`Astaroth` scales to thousands of GPUs [@pekkila2022scalable], and has an ergonomic DSL that can be used to rewrite existing PDE solvers and to write completely new ones.

`Astaroth` was originally created to run astrophysical plasma simulations on GPUs.
A widely used library for astophysical plasma simulations is the Pencil Code [@brandenburg2020pencil], which is a modular PDE solver for compressible hydrodynamics.
`Astaroth` has successfully been used to accelerate it [@puro2023programmatic], with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and convolutions, are traditionally expressed using stencils.
`Astaroth` enables this task by cleanly separating the front-end (DSL) from the back-end (compiler and runtime), which also provides researchers with a platform for performance research.

# State of the field                                                                                                                  

Compared to existing DSL approaches for stencil computations [@ragan2013halide; mullapudi2015polymage] `Astaroth` specializes in cache-constrained computations required for 3D multi-physics simulations, which run out of the available cache due to the need of having many interdependent values in working memory at the same time.

Some other common tools for acceleration of scientific tools include `Kokkos` [@trott2021kokkos], `Raja` [@beckingsale2019raja], `OpenMP` [@dagum1998openmp] and `OpenACC` [@openacc2025spec]. 
Compared to these `Astaroth` gives more control to the user, and especially when compared to the directive based approaches of `OpenMP` and `OpenACC`, it is less of a black-box solution. Furthermore, `Astaroth` handles a larger scope of responsibilities for the application codes, like handling the required communications for multi-process applications and automatic scheduling of computational tasks. Compared to the C++ based approaches of `Kokkos` and `Raja` `Astaroth` exposes a more language agnostic API, extending the range of applications which can take advantage of `Astaroth`. Furthermore, all of these approaches are general frameworks while `Astaroth` tries to modify itself to the application at hand to the best of its abilities, as expanded upon on later. 

# Software design

`Astaroth` consists of four main components: 1) a domain-specific language (DSL) for defining stencil computations, 2) an API for executing them on multi-GPU platforms, and 3) a standalone solver for running programs written using the DSL.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## DSL compiler and runtime API

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to deal with technical implementation details.
Stencils are written in a declarative syntax, and compute kernels that use them are written in an imperative syntax.
As stencils are declarative, their implementation is left up to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized stencil optimizations.
This frees the user from understanding how to optimize stencil data access operations on GPUs.
The DSL compiler transpiles the DSL source into CUDA or HIP source, which is compiled into machine code using a native CUDA or HIP compiler.
The program produced by the DSL compiler is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread group sizes for kernel execution,

## Multi-GPU runtime API

`Astaroth` has a multi-GPU runtime which supports defining directed acyclic graphs (DAGs) of compute kernels, halo exchange operations, boundary conditions, and special operations such as reductions.
These DAGs are called `TaskGraphs` internally.
A task scheduler executes any number of iterations of the `TaskGraph` as data dependencies are satisfied.
GPU-to-GPU remote direct memory access (RDMA) is used for faster data transfer.
For platforms that do not support GPU-to-GPU RDMA, a slower CPU-to-CPU communication method is also provided.

`Astaroth` provides an API with foreign function interoperability for accessing this runtime.
The API is organized into two layers: the `Device` layer and the `Grid` layer.
The `Device` layer provides access to single-GPU functionality, such as loading and storing data, launching kernels, applying boundary conditions, and loading/storing snapshots from disk.
The `Grid` layer provides access to multi-GPU functionality, such as running `TaskGraphs`, distributed initialization, and distributed loading/storing of snapshots.
Other special functionality is also provided through the API, such as fourier transforms, reduction operations, and 2D-slice output.

## Solver

 - `Astaroth` also includes a standalone solvera
 - OL: is this the `standalone_mpi` solver? I read through the source, and it only supports four hard coded simulation cases: MHD, shock, hydro_heatduct, and bound_test. If this is what is meant by the standalone solver, I think a better case is madwe by focusing either on the MHD solver specifically, or mention the PCA work.

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly in astrophysical settings involving magnetohydrodynamic turbulence [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. Additionally it has been used in different methods papers focusing on performance [@pekkila2022scalable; @pekkila2017methods; @yokelson2024soma; @pekkila2025stencil; @puro2025gpu].
The aforementioned acceleration of `Pencil Code`  is expected to increase the number of people relying on `Astaroth` as the core execution engine. The associated performance increase will enable more realistic astrophysical simulations in a wide range of scientific applications from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution, who include Jörn Warnecke, Frederick Gent, Indrani Das, Ruben Krasnopolsky.

Would Maarit know the best which funding sources to cite for the development of Astaroth??

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém, Tzu-Chun Hsu and Jack Hsu.
