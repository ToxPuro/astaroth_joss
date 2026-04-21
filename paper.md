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
In recent years, GPUs have become the primary compute platform for data-parallel applications high-performance computing, and it is ~~difficult to run large simulations without them~~.
> JP comment in source below
<!--
- JP: "difficult" -> can also run large simulations with CPUs. Suggest something like "the leap in parallel throughput is needed to reach state-of-the-art accuracy in computational physics simulations"
-->
`Astaroth` is a GPU framework for stencil computations, that has been developed with scalable scientific computing in mind.

`Astaroth` provides its own domain specific language (DSL), in which researchers can express their required computations without having to focus on technical implementation details.
It can run efficiently both on CUDA- and HIP-based environments --- and even on hardware lacking GPUs, e.g. for testing purposes.
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing and has library integrations for performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
To further ease the development and acceleration of PDE solvers based on the finite-difference method, `Astaroth` comes together with its own PDE solver and a standard library with support for a variety of PDE-solver functionality.
`Astaroth` has primarily been used for turbulent astrophysical plasma simulations.

# Statement of need

Much of the software used for scientific computing is written for CPUs, and has to be ported to GPUs to run larger problems.
`Astaroth` has been developed to solve this problem for the subset of scientific software that can be expressed as stencil computations.
`Astaroth` ~~scales~~ to thousands of GPUs [@pekkila_graphicsprocessors_2026], and has a high-level DSL that can be used to rewrite existing PDE solvers and to write completely new ones.
> JP comment in source below
<!--
%JP: 
**scales**
- better to use the thesis (2022 paper only goes up to 64 devices, in the thesis we do 4096).
- "scales" -> has been demonstrated to scale weakly at ?% efficiency to 64 devices in MHD simulations (2022 paper, could leave this out and focus on the TFM case). Has been demonstrated to scale weakly at 93% efficiency to 4096 devices in computations with the test-field method. 
-->

> JP comment in source below
<!--
%JP: some notes if we're going to include history and our prior work
% Astaroth Code: hard-coded compressible hydrodynamics 2014-2019[@vaisala_magneticphenomena_2017;pekkila_methodscompressible_2017]
% Astaroth: generalized stencil framework 2019- (DSL V1, single-gpu) [@pekkila_masters_2019], rewritten DSL V2 grammar and code generator [@pekkila_stencilcomputations_2025]
% Astaroth: single-node multi-gpu[@vaisala_interactionlarge_2021], general MPI halo exchange communication and Z-order [@pekkila2022scalable], distributed reductions(lappi, not sure if mentioned in the thesis) and task system[@lappi], topology-aware domain decomposition and mapping and communication optimizations (fused packing) and TFM[@pekkila_graphicsprocessors_2026]
% Astaroth: CPU, PC-A, DSL improvements, ray tracing, and other contributions [@Touko'sWork]
-->
`Astaroth` was originally created to run astrophysical plasma simulations on GPUs.
A widely used library for astophysical plasma simulations is the Pencil Code [@brandenburg2020pencil], which is a modular PDE solver for compressible hydrodynamics.
`Astaroth` has successfully been used to accelerate it [@puro2023programmatic], with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and convolutions, are traditionally expressed using stencils.
`Astaroth` enables this task by cleanly separating the front-end (DSL) from the back-end (compiler and runtime), which also provides researchers with a platform for performance research.

> JP comment in source below
<!--
%JP SOME REFERENCES START
@phdthesis{vaisala_magneticphenomena_2017,
 address = {Helsinki, Finland},
 author = {V{\"a}is{\"a}l{\"a}, M. S.},
 note = {\url{https://urn.fi/URN:ISBN:978-951-51-2778-5}},
 school = {University of Helsinki},
 title = {Magnetic Phenomena of the Interstellar Medium in Theory and Observation},
 year = {2017}
}
% Väisälä, Miikka S., et al. “Interaction of Large- and Small-Scale Dynamos in Isotropic Turbulent Flows from GPU-Accelerated Simulations.” The Astrophysical Journal, vol. 907, no. 2, Feb. 2021, p. 83. DOI.org (Crossref), https://doi.org/10.3847/1538-4357/abceca.
%%%%%%%%%%%%%%%%%%%%%%%%%% JP SOME REFERENCES END
-->

# State of the field                                                                                                                  
> JP comment in source below
<!--
%%%% JP revised suggestion START
-->
Methods to achieve performance portability in stencil computations have been widely studied.
Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[@mullapudi2015polymage].
Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
More generalized software projects that provide the building blocks for domain-specialized libraries have also been proposed.
Delite[@sujeeth_delitecompiler_2014] and Lift[@steuwer_liftfunctional_2017] provide intermediate languages as targets for domain-specific languages. <!--% JP (can be left out if no room) -->
Kokkos[@trott2021kokkos] and RAJA[@beckingsale2019raja] provide abstraction layers for parallel computational patterns but focus on single-node computations.
The Chapel[@callahan_cascadehigh_2004] and Charm++[@kale_charmportable_1993] provide programming models for parallel and distributed computations.
In a more specialized approach, the Cactus framework[@goodale_cactusframework_2003] provides a collection of functionalities shared between computational science tasks.
We refer the reader to [@pekkila_graphicsprocessors_2026] for more details on the background.

Closest to Astaroth is Parthenon[@grete_parthenonperformance_2023], which is a distributed framework for adaptive mesh refinement using Kokkos as the backend for intra-node computations.
However, Astaroth provides a DSL and an optimizing code generator for implementing the computations akin to Halide, Polymage, and Patus with a focus on structured-grid computations.
Astaroth also incorporates other key functionalities for computational sciences, e.g., distributed reductions, IO, and different physics cases in a modular manner.
A distinctive feature of Astaroth is its specialization for cache-constrained use cases, especially in multiphysics simulations where interdependent values need to be held in working memory at the same time.
> JP comment in source below (bibliography additions)
<!--
@article{sujeeth_delitecompiler_2014,
 author = {Sujeeth, A. K. and Brown, K. J. and Lee, H. and Rompf, T. and Chafi, H. and Odersky, M. and Olukotun, K.},
 doi = {10.1145/2584665},
 journal = {{ACM} Transactions on Embedded Computing Systems},
 number = {4s},
 pages = {1--25},
 title = {Delite: A Compiler Architecture for Performance-Oriented Embedded Domain-Specific Languages},
 volume = {13},
 year = {2014}
}
@inproceedings{steuwer_liftfunctional_2017,
 author = {Steuwer, M. and Remmelg, T. and Dubach, C.},
 booktitle = {Proceedings of the 2017 {IEEE}/{ACM} International Symposium on Code Generation and Optimization},
 doi = {10.1109/CGO.2017.7863730},
 pages = {74--85},
 publisher = {{IEEE}},
 title = {{LIFT}: A Functional Data-Parallel {IR} for High-Performance {GPU} Code Generation},
 year = {2017}
}
@inproceedings{callahan_cascadehigh_2004,
 author = {Callahan, D. and Chamberlain, B. L. and Zima, H. P.},
 booktitle = {Proceedings of the Ninth International Workshop on High-Level Parallel Programming Models and Supportive Environments, 2004. Proceedings.},
 doi = {10.1109/HIPS.2004.1299190},
 pages = {52--60},
 publisher = {{IEEE}},
 title = {The Cascade High Productivity Language},
 year = {2004}
}
@inproceedings{kale_charmportable_1993,
 author = {Kal{\'e}, L. V. and Krishnan, S.},
 booktitle = {Proceedings of the Eighth Annual Conference on Object-Oriented Programming Systems, Languages, and Applications},
 doi = {10.1145/165854.165874},
 pages = {91--108},
 publisher = {{ACM}},
 title = {{CHARM}++: A Portable Concurrent Object Oriented System Based on {C}++},
 year = {1993}
}
%%%% JP revised suggestion END
-->

> JP comment in source below
<!--
% JP: commented out the previous version
Methods to accelerate and improve perfomance-portability and productivity of stencil computations are widely studied. We refer the reader to [pekkila_graphicsprocessors_2026] for more details on the background.
There are widely used lower-level tools such as OpenMP[dagum1998openmp], OpenACC[openacc2025spec], Kokkos[@trott2021kokkos] and Raja[@beckingsale2019raja] which provide abstraction layers for parallel computational patterns, but which still leave e.g. performing the required communications to the user. 
Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[mullapudi2015polymage]. Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
Closest to Astaroth are domain-specialized frameworks providing the building blocks for computational physics, such as Cactus[@goodale_cactusframework_2003] and Parthenon[@grete_parthenonperformance_2023].
Compared to these projects, `Astaroth` uses a DSL, resembling the approaches of Halide, Polymage, and Patus for their domains.
Distinctively to existing approaches, `Astaroth` specializes, and has extensive optimizations, for cache-constrained use cases, such as multiphysics simulations which need many interdepent values in working memory at the same time. Additionally, `Astaroth` incorporates other needed operations, like reductions, together with stencils to cover use cases that are not fully stencil-based.


%%%%% JP START: added bullet points of things %%%%%%%%%%%%%
- methods to accelerate and improve perfomance-portability and productivity Stencil computations widely studied: numerous software projects at different levels of abstractions. We refer the reader to [pekkila_graphicsprocessors_2026] for more details on the background.
- Low-level generalized approaches suitable for implementing stencil solvers for graphics processors include CUDA and HIP [@nvidia_cudac_2025;@amd_hipdocumentation_2024] (computation) and GPU-aware MPI[messagepassinginterfaceforum_openmpi_2025;argonnenationallaboratoryandmississippistateuniversity_mpich_2026] (communication).
- OpenMP[dagum1998openmp], OpenACC[openacc2025spec], Kokkos[@trott2021kokkos], Raja[@beckingsale2019raja] provide abstractions and portability layers for parallel computational patterns and are suitable for implementing domain-specialized solvers on shared-memory systems.
- Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[mullapudi2015polymage]. Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
- Closest to Astaroth are domain-specialized frameworks providing the building blocks for computational physics, such as Cactus[@goodale_cactusframework_2003] and Parthenon[@grete_parthenonperformance_2023].
- Closest to our work is the Parthenon, which provides a framework for distributed computations of  adaptive mesh refinement, utilizing Kokkos as the backend for shared-memory computation.
- Like these projects, Astaroth provides ready-made components for implementing computation and communication for their domain-specialized case.
- Compared to these projects, Astaroth uses a domain-specific language that is lowered into platform-portable autotuned CUDA/HIP code to implement efficient stencil computations, resembling the approaches of Halide, Polymage, and Patus for their domains.
- The distinctive feature of Astaroth is that it specializes in cache-constrained computations on structured grids especially in multiphysics simulations, where the coupling of physical fields, the use of large stencils, and the use of double-precision arithmetic. These requirements result in prohibitively large working sets in on-chip memory for obtaining sufficient levels of reuse to break through the memory bandwidth bound. Astaroth implements a code generator for unrolling and reordering the stencil computations to reduce instruction counts, improve the locality of memory accesses, and ensuring sufficient instruction-level parallelism to obtain improved latency-hiding at low occupancy due to high register allocation per thread to improve the reuse of intermediate values.

%%%%%%%%%%%%%%%%%%%%%% JP END

### COMMENT (Touko)

**Made now a draft of the body text based on Johannes' bullet points. Johannes, thoughts? Please edit if you don't like it.**
**JP:** OK. Revised above (the uncommented one).
-->


# Software design

`Astaroth` consists of three main components: 1) `acc`, a compiler and runtime for a domain-specific language (DSL) for stencil computations, 2) an API for executing stencil applications on multi-GPU platforms, and 3) a standalone solver for certain simulation cases.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## `acc` compiler and runtime

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to deal with technical implementation details.
The main operations, like stencils, are **declarative**, meaning only the desired results are specified and ther implementations is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. Rest of the code using these operations is imperative.
> JP comment in source below
<!--
%JP: declarative/imperative ambiguous and hard to understand here. Suggest: "imperative stream programming language with reduced/simplified/minimal syntax with a language feature for specifying coefficients for linear stencil operations" or similar
%JP: Could also consider specifying our use of declarative/imperative in this context first if the word count is not a limit or moving the "..their implementation is left up..." paragraph up.
-->
> TP answer to above
<!--
TP: agreed. Moved the section now up.
-->
In addition to stencils the DSL supports reductions -- which are commonly needed for stencil-based solvers and require multiple steps to perform across multiple GPUs -- and distibuted ray-tracing along integer coordinate lines -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].

Abstracting away these distributed GPU-application optimizations reduces the complexity at the DSL source level.
`acc` transpiles the DSL source into CUDA or HIP source, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc-runtime`, which further optimizes the kernels by autotuning the thread group sizes for kernel execution.


Sometimes values of variables change the branches taken in a particular kernel. Not knowing which branches are taken limits `Astaroth`'s ability to optimize the code severely.
To know which branches are taken, `Astaroth` supports run-time compilation, which is also used to eliminate unused code and variables.
> JP comment in source below
<!--
- unclear how run-time compilation and branches are related. We do the CPU compilation first and assume all ifs are true. Is there some other mechanism that triggers recompilation during the actual simulation?
-->
> TP answer to above
<!--
  We do run-time compilation exactly so we do not have to assume all ifs are true (this is hardly the case for large simulation codes). Also the CPU analysis code is recompiled so it can known which branches to take. Recompilation is triggered explicitly by the user by an API function.
Modified so the text explains this more directly.
-->

### COMMENT (Touko)

**Agreed. Tried to reword it now to be more concise and clear**

**Will expand here on the issue so maybe we come to conclusion how to further improve it.**
**The issue is two-fold: Firstly, Astaroth needs to know which stencils are called at compile-time since it takes each stencil and writes out the computation in full at the start of the kernels. This does not play nicely with conditionals for two reasons: either redundant computations will be performed or even more catastrophically some stencils are missed to be generated all together. This is because Astaroth uses execution information to gather information about which stencils are called. The second issue is less of a showstopper but still needs to be addressed: too much code. Translating PC to Astaroth will produce kernels with > 10,000 lines of code. Much of this code is redundant since we know, since we now know the values of all relevant variables, that a large majority of the code is never executed. So we eliminate this. One could think that it would be sufficient to make all the variables constexpr and let the CUDA/HIP-compilers handle it but that at least the drawback that it makes compilation take in the order of one hour which is somewhat ridiculous (okay this experiment was some time ago so might not be exactly true anymore, but you get the gist)**

**So these two problems motivate runtime-compilation.**


### COMMENT (Oskar) (Deleted the previous comment for brevity)

**Ok, it seems like both points are about conditional compilation**

**To summarize: Data about stencil usage in kernels is used for conditional compilation. I assume by "execution information" you mean data dependencies deduced from the stencils.**
**I rewrote the paragraph based on this understanding, let me know if I missed something. I left out the detail about improved CUDA/HIP compilation performance, as that feels like a technical detail. I also feel like raising attention to a problem that users would expect to be solved (long compilation times) doesn't particularly help sell Astaroth either**

### COMMENT (Touko) 

**By execution information I mean information gathered by executing a specialized CPU analysis version of the code**
**This information is used quite extensively for any possible purposes: stencils,reductions,ray-tracing,communication,optimizing variables out etc..**
**Tried to reword the paragraph to capture this more general fact**



## Multi-GPU runtime API

`Astaroth` has a multi-GPU runtime supporting directed acyclic graphs (DAGs) of kernel calls, halo exchange operations and boundary conditions.
A DAG can be defined in the DSL using a `ComputeSteps` declaration, specifying which kernels to call, and which boundary conditions to impose.
These compute steps are decomposed into data regions, with spatial data dependencies coming from the used stencils.
A task scheduler executes any number of iterations of the DAG as data dependencies are satisfied, and is free to reorder the operations as long as dependency relationships are not violated.
This asynchronous scheduling improves performance in communication-bound cases, especially for higher process counts \cite{lappi2021task}.
For fast data transfers and to support all possible hardware, both GPU-to-GPU remote direct memory access (RDMA) and CPU-to-CPU communication are supported.

`Astaroth`'s runtime API is C-ABI compatible, supporting foreign function interfaces in external applications written in any programming language.
The API is organized into two layers: the `Device` layer and the `Grid` layer.
The `Device` layer provides access to single-GPU functionality, such as moving data between CPU and GPU, launching kernels, and loading/storing snapshots from disk.
The `Grid` layer provides access to multi-GPU functionality, such as executing DAGs, distributed initialization, and distributed loading/storing of snapshots.
Other special functionality is also provided through the API, such as distributed Fourier transforms.

## Solver

`Astaroth` also includes a standalone solver, which can be used to write new simulations and as a simple test case for performance research.
`acc-runtime/samples/mhd_modular` contains a baseline MHD solver, which can either be used as is or be extended to cover more physics.

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
