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
 
date: 15 April 2026
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

One of the predecessors of `Astaroth` is the Pencil Code [@brandenburg2020pencil], which is a widely used modular PDE solver for compressible hydrodynamics.
`Astaroth` has successfully been used to accelerate it [@puro2023programmatic], with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and convolutions, are traditionally expressed using stencils.
`Astaroth` enables this task by cleanly separating the front-end (DSL) from the back-end (compiler and runtime), which also provides researchers with a platform for performance research.

# State of the field                                                                                                                  

Compared to existing DSL approaches for stencil computations [@ragan2013halide; mullapudi2015polymage] `Astaroth` specializes in cache-constrained[^cache_def] computations required for 3D multi-physics simulations, which run out of the available cache due to the need of having many interdependent values in working memory at the same time.

Some other common tools for acceleration of scientific tools include `Kokkos` [@trott2021kokkos], `Raja` [@beckingsale2019raja], `OpenMP` [@dagum1998openmp] and `OpenACC` [@openacc2025spec]. 
Compared to these `Astaroth` gives more control to the user, and especially when compared to the directive based approaches of `OpenMP` and `OpenACC`, it is less of a black-box solution. Furthermore, `Astaroth` handles a larger scope of responsibilities for the application codes, like handling the required communications for multi-process applications and automatic scheduling of computational tasks. Compared to the C++ based approaches of `Kokkos` and `Raja` `Astaroth` exposes a more language agnostic API, extending the range of applications which can take advantage of `Astaroth`. Furthermore, all of these approaches are general frameworks while `Astaroth` tries to modify itself to the application at hand to the best of its abilities, as expanded upon on later. 

# Software design

`Astaroth`'s design is too expansive to cover here fully. Instead we give what we believe to be the four main principles of the design. For further information on the design the reader can refer to the documentation of `Astaroth`.

1): Emphasis on inter-operability.
`Astaroth`'s external API is mostly in C for easy interoperatibility from any programming language and has always been designed to be called from external applications. 
2): Specialization to the use case at hand.
The whole library is always compiled from scratch based on the user's DSL code, providing the largest amount of specialization to the use case at hand. This specialization enables high performance and an ergonomic API. This principle is taken further by compiling the library dynamically during the runtime of the application, allowing specialization to complex dynamical situations. As much as possible, this specialization is driven by automatic inference done by the DSL compiler and runtime. This allows `Astaroth` to uncover important optimizations in large code-bases achieving close to hand-tuned performance in cases where the amount of code prohibits specialization done by the user.
3): Ergonomic and high performance via declarativity.
While the DSL itself follows C/C++ closely, the most important objects, the stencils themselves, are declarative in nature. This makes their usage ergonomic and gives the compiler enough freedom to choose the most performant way to perform the required computations.
Similar design ethos carries to higher-level components of `Astaroth`.  A good example of this are `ComputeSteps`; The user describes steps of computations and `Astaroth` handles all execution details of performing the computations such as the required communications with multiple GPUs and chooses the most performant way to achieve the results.
4): Multi-layered API.
`Astaroth` has also more explicit APIs which the more declarative APIs take advantage of internally. This is important so the user can choose the correct abstraction level for their use case and for `Astaroth` as a platform for performance research.

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly in astrophysical settings involving magnetohydrodynamic turbulence [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. Additionally it has been used in different methods papers focusing on performance [@pekkila2022scalable; @pekkila2017methods; @yokelson2024soma; @pekkila2025stencil; @puro2025gpu].
The aforementioned acceleration of `Pencil Code`  is expected to increase the number of people relying on `Astaroth` as the core execution engine. The associated performance increase will enable more realistic astrophysical simulations in a wide range of scientific applications from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution, who include Jörn Warnecke, Frederick Gent, Indrani Das, Ruben Krasnopolsky and Hsien Shang.

Would Maarit know the best which funding sources to cite for the development of Astaroth??

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^cache_def]: For the uninitiated cache refers to the memory closest to the execution units of computers, used to store frequently accessed data. The handling of it often becomes the most peformance-sensitive part of computations, due to the slowness of memory reads to slower memory.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém, Tzu-Chun Hsu and Jack Hsu.
