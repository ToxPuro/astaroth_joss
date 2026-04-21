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
`Astaroth` scales to thousands of GPUs [@pekkila2022scalable], and has a high-level DSL that can be used to rewrite existing PDE solvers and to write completely new ones.

`Astaroth` was originally created to run astrophysical plasma simulations on GPUs.
A widely used library for astophysical plasma simulations is the Pencil Code [@brandenburg2020pencil], which is a modular PDE solver for compressible hydrodynamics.
`Astaroth` has successfully been used to accelerate it [@puro2023programmatic], with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and convolutions, are traditionally expressed using stencils.
`Astaroth` enables this task by cleanly separating the front-end (DSL) from the back-end (compiler and runtime), which also provides researchers with a platform for performance research.

# State of the field                                                                                                                  
%%%%% JP START: added bullet points of things %%%%%%%%%%%%%
- methods to accelerate and improve perfomance-portability and productivity Stencil computations widely studied: numerous software projects at different levels of abstractions. We refer the reader to [pekkila_graphicsprocessors_2026] for more details on the background.
- Low-level generalized approaches suitable for implementing stencil solvers for graphics processors include CUDA and HIP [@nvidia_cudac_2025;@amd_hipdocumentation_2024] (computation) and GPU-aware MPI[messagepassinginterfaceforum_openmpi_2025;argonnenationallaboratoryandmississippistateuniversity_mpich_2026] (communication).
- OpenMP[dagum1998openmp], OpenACC[openacc2025spec], Kokkos[@trott2021kokkos], Raja[@beckingsale2019raja] provide abstractions and portability layers for parallel computational patterns and are suitable for implementing domain-specialized solvers on shared-memory systems.
- Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[mullapudi2015polymage]. Autotuning code-generation frameworks includes Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
- Closest to Astaroth are domain-specialized frameworks providing the building blocks for computational physics, such as Cactus[@goodale_cactusframework_2003] and Parthenon[@grete_parthenonperformance_2023].
- Closest to our work is the Parthenon, which provides a framework for distributed computations of  adaptive mesh refinement, utilizing Kokkos as the backend for shared-memory computation.
- Like these projects, Astaroth provides ready-made components for implementing computation and communication for their domain-specialized case.
- Compared to these projects, Astaroth uses a domain-specific language that is lowered into platform-portable autotuned CUDA/HIP code to implement efficient stencil computations, resembling the approaches of Halide, Polymage, and Patus for their domains.
- The distinctive feature of Astaroth is that it specializes in cache-constrained computations on structured grids especially in multiphysics simulations, where the coupling of physical fields, the use of large stencils, and the use of double-precision arithmetic. These requirements result in prohibitively large working sets in on-chip memory for obtaining sufficient levels of reuse to break through the memory bandwidth bound. Astaroth implements a code generator for unrolling and reordering the stencil computations to reduce instruction counts, improve the locality of memory accesses, and ensuring sufficient instruction-level parallelism to obtain improved latency-hiding at low occupancy due to high register allocation per thread to improve the reuse of intermediate values.

@phdthesis{pekkila_graphicsprocessors_2026,
 address = {Espoo, Finland},
 author = {Pekkilä, Johannes},
 keywords = {Graphics processors, high-performance computing, computational astrophysics},
 note = {\url{https://urn.fi/URN:ISBN:978-952-64-3160-4}},
 school = {Aalto University School of Science},
 title = {Graphics processors in high-performance computing: Practice and experience in the simulation of astrophysical magnetohydrodynamics and turbulence},
 year = {2026}
}
@manual{nvidia_cudac_2025,
 author = {{Nvidia}},
 edition = {V12.4},
 note = {\url{https://web.archive.org/web/20240521004520/https://docs.nvidia.com/cuda/pdf/CUDA_C_Programming_Guide.pdf}},
 title = {{CUDA} {C}++ Programming Guide},
 year = {2025}
}
@manual{amd_hipdocumentation_2024,
 author = {{AMD}},
 edition = {V6.1.0},
 note = {\url{https://web.archive.org/web/20240529081240/https://rocm.docs.amd.com/_/downloads/HIP/en/docs-6.1.0/pdf/}},
 title = {{HIP} Documentation},
 year = {2024}
}
@manual{messagepassinginterfaceforum_openmpi_2025,
 author = {Message Passing Interface Forum},
 edition = {V5.0.9},
 note = {\url{https://web.archive.org/web/20260212014306/https://docs.open-mpi.org/en/v5.0.9/}},
 title = {Open {MPI}},
 year = {2025}
}
@manual{argonnenationallaboratoryandmississippistateuniversity_mpich_2026,
 author = {Argonne National Laboratory and Mississippi State University},
 edition = {V5.0.0},
 note = {\url{https://web.archive.org/web/20260316064836/https://github.com/pmodels/mpich}},
 title = {{MPICH}},
 year = {2026}
}
@inproceedings{christen_patuscode_2011,
 author = {Christen, M. and Schenk, O. and Burkhart, H.},
 booktitle = {Proceedings of the 2011 {IEEE} International Parallel \& Distributed Processing Symposium},
 doi = {10.1109/IPDPS.2011.70},
 pages = {676--687},
 publisher = {{IEEE}},
 title = {{PATUS}: A Code Generation and Autotuning Framework for Parallel Iterative Stencil Computations on Modern Microarchitectures},
 year = {2011}
}
@article{lutz_partansautotuning_2013,
 author = {Lutz, T. and Fensch, C. and Cole, M.},
 doi = {10.1145/2400682.2400718},
 journal = {{ACM} Transactions on Architecture and Code Optimization},
 number = {4},
 pages = {59:1--59:24},
 title = {{PARTANS}: An Autotuning Framework for Stencil Computation on Multi-{GPU} Systems},
 volume = {9},
 year = {2013}
}
@incollection{goodale_cactusframework_2003,
 author = {Goodale, T. and Allen, G. and Lanfermann, G. and Mass{\'o}, J. and Radke, T. and Seidel, E. and Shalf, J.},
 booktitle = {High Performance Computing for Computational Science --- {VECPAR} 2002},
 doi = {10.1007/3-540-36569-9_13},
 editor = {Goos, G. and Hartmanis, J. and Van Leeuwen, J. and Palma, J. M. L. M. and Sousa, A. A. and Dongarra, J. and Hern{\'a}ndez, V.},
 pages = {197--227},
 publisher = {{Springer}},
 title = {The Cactus Framework and Toolkit: Design and Applications},
 year = {2003}
}
@article{grete_parthenonperformance_2023,
 author = {Grete, P. and Dolence, J. C. and Miller, J. M. and Brown, J. and Ryan, B. and Gaspar, A. and Glines, F. and Swaminarayan, S. and Lippuner, J. and Solomon, C. J. and Shipman, G. and Junghans, C. and Holladay, D. and Stone, J. M. and Roberts, L. F.},
 doi = {10.1177/10943420221143775},
 journal = {The International Journal of High Performance Computing Applications},
 number = {5},
 pages = {465--486},
 title = {Parthenon---a Performance Portable Block-Structured Adaptive Mesh Refinement Framework},
 volume = {37},
 year = {2023}
}
%%%%%%%%%%%%%%%%%%%%%% JP END


Compared to existing DSL approaches for stencil computations [@ragan2013halide; mullapudi2015polymage] `Astaroth` specializes in cache-constrained computations required for 3D multi-physics simulations, which run out of the available cache due to the need of having many interdependent values in working memory at the same time.

Some other common technologies used by computational scientists to add GPU support include `Kokkos` [@trott2021kokkos], `Raja` [@beckingsale2019raja], `OpenMP` [@dagum1998openmp] and `OpenACC` [@openacc2025spec]. 
Compared to these technologies, `Astaroth` is specialized for a particular problem (stencil computation). Because of this more constrained domain, `Astaroth` can take care of stencil optimization, communication, scheduling, and autotuning.

# Software design

`Astaroth` consists of three main components: 1) a domain-specific language (DSL) for defining stencil computations, 2) an API for executing them on multi-GPU platforms, and 3) a standalone solver for running programs written using the DSL.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## DSL compiler and runtime API

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to deal with technical implementation details.
Stencils are written in a declarative syntax, and kernels that use them are written in an imperative syntax.
As stencils are declarative, their implementation is left up to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized stencil optimizations.
This frees the user from understanding how to optimize stencil data access operations on GPUs.
The DSL compiler transpiles the DSL source into CUDA or HIP source, which is compiled into machine code using a native CUDA or HIP compiler.
The program produced by the DSL compiler is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread group sizes for kernel execution.

Additionally the DSL has support for certain special operations.
Reductions, which usually require multiple steps to perform across multiple GPUs, are written as declarative statements, and are optimized along the stencil computations.
The DSL also supports distibuted ray-tracing along integer coordinate lines, which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].



To achieve good performance it is important that for each kernel it is known which stencils are called and how are they called. 
This is restrictive for simulation codes having a large amount of control-flow which depends on dynamically chosen variables. Thus `Astaroth` supports dynamic compilation of the whole library, thanks to which the dynamic variables can be treated as if they were known at compile-time.
Additionally, because of this `acc` provides code elimination which removes unused control-flow by leveraging the known values of the variables.

### COMMENT (Oskar): the above paragraph is a bit confusing, I would like to edit it for readability, but I'm not sure what it is trying to say.

**Is the paragraph talking about data dependencies? ("each kernel ... [knows] which stencils are called and how") or constant folding and conditional compilation? ("large amount of control-flow", "`acc` provides code elimination which removes unused control-flow").
Needs cleanup.**


## Multi-GPU runtime API

`Astaroth` has a multi-GPU runtime which supports defining directed acyclic graphs (DAGs) of kernel calls, halo exchange operations and boundary conditions. These DAGs, which are called `TaskGraphs`, are defined as ordered steps to be performed in the language construct called `ComputeSteps`. Only the kernel calls and which boundary conditions to be applied are specified and everything else is deduced from the dependencies. Thus, similar to stencils the syntax is declarative, which enables `Astaroth` to handle the details and to apply any possible optimizations.
A task scheduler executes any number of iterations of the `TaskGraphs` as data dependencies are satisfied, which enables an increased amount of overlap of communication and computation.
For fast data transfers and to support all possible hardware, both GPU-to-GPU remote direct memory access (RDMA) and CPU-to-CPU communication are supported.

`Astaroth` provides an API with foreign function interoperability for accessing this runtime.
The API is organized into two layers: the `Device` layer and the `Grid` layer.
The `Device` layer provides access to single-GPU functionality, such as loading and storing data, launching kernels, and loading/storing snapshots from disk.
The `Grid` layer provides access to multi-GPU functionality, such as running `TaskGraphs`, distributed initialization, and distributed loading/storing of snapshots.
Other special functionality is also provided through the API, such as distributed Fourier transforms.

## Solver

`Astaroth` also includes a standalone solver, which can be used to write new simulations and to get familiar with `Astaroth`.
`acc-runtime/samples/mhd_modular` contains a baseline MHD solver, which can either be used as is or be extended to cover more physics.
It also works as a simple test case for performing performance research.

 - OL: is this the `standalone_mpi` solver? I read through the source, and it only supports four hard coded simulation cases: MHD, shock, hydro_heatduct, and bound_test. If this is what is meant by the standalone solver, I think a better case is madwe by focusing either on the MHD solver specifically, or mention the PCA work.
 - TP: yes this is the standalone solver. The source code is horribly out of date but it is not as limited as the source code makes it out to be (the PhysicsSimulation Enums do not need to be used). And can be relatively easily extended to cover more cases, which exactly what we are doing with the Taiwanese.

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. Additionally it has been used in different methods papers focusing on performance [@pekkila2022scalable; @pekkila2017methods; @yokelson2024soma; @pekkila2025stencil; @puro2025gpu].
The aforementioned acceleration of `Pencil Code`  is expected to increase the number of people relying on `Astaroth` as the core execution engine. The associated performance increase will enable more realistic astrophysical simulations in a wide range of scientific applications from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution, who include Jörn Warnecke, Frederick Gent, Indrani Das, Ruben Krasnopolsky.

Would Maarit know the best which funding sources to cite for the development of Astaroth??

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém, Tzu-Chun Hsu and Jack Hsu.
