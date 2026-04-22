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
> READ THIS!!: Software design-section is under the dictatorship of Oskar, so please do not directly edit it. Instead if you want to propose changes add them in comments like this. The other sections are under the tentative dictatorship of Touko, who is fine with you editing the text directly, but who might revert your edits if they make the text worse.

> OL: Remember to leave a line gap between comments like this, so that they are not smushed together

Stencil computations[^stencil_footnote] are one of the bedrocks of high-performance scientific simulations, forming the core of many partial differential equation (PDE) and numerical linear algebra solvers. 
In recent years, GPUs have become the primary compute platform for data-parallel applications in high-performance computing, and it is difficult to run large simulations without them.
> JP comment in source below
<!--
- JP: "difficult" -> can also run large simulations with CPUs. Suggest something like "the leap in parallel throughput is needed to reach state-of-the-art accuracy in computational physics simulations"
- TP: I would agree with Oskar's wording that difficult is an apt phrasing. GPUs are never strictly needed, but clearly it is easier to run larger simulations with them.
-->
To address this, `Astaroth` is a GPU framework for stencil computations, that has been developed with scalable scientific computing in mind.

`Astaroth` provides its own domain specific language (DSL), in which researchers can express their required computations without having to focus on technical implementation details.
It can run efficiently both on CUDA- and HIP-based environments --- and even on hardware lacking GPUs, e.g. for testing purposes.
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing and has library integrations for performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
To further ease the development and acceleration of PDE solvers based on the finite-difference method, `Astaroth` comes together with its own PDE solver and a standard library with support for a variety of PDE-solver functionality.
`Astaroth` has primarily been used for turbulent astrophysical plasma simulations.

# Statement of need

Much of the software used for scientific computing is written for CPUs, and has to be ported to GPUs to run larger problems.
`Astaroth` has been developed to solve this problem for the subset of scientific software that can be expressed as stencil computations.
`Astaroth` scales (with a weak scaling efficiency of greater than 90%) to thousands of GPUs [@pekkila_graphicsprocessors_2026], and has a high-level DSL that can be used to rewrite existing PDE solvers and to write completely new ones.
> JP comment in source below
<!--
%JP: 
**scales**
- better to use the thesis (2022 paper only goes up to 64 devices, in the thesis we do 4096).
- "scales" -> has been demonstrated to scale weakly at ?% efficiency to 64 devices in MHD simulations (2022 paper, could leave this out and focus on the TFM case). Has been demonstrated to scale weakly at 93% efficiency to 4096 devices in computations with the test-field method. 
%TP:
Made the statement now more precise by having the weak scaling efficiency in brackets. What do you think does that suffice? If the reader cares about details they can lookup Johannes' thesis.
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
A distinctive feature of Astaroth is its specialization for cache-constrained use cases, especially in multiphysics simulations where interdependent values need to be held in working memory at the same time. Additionally, `Astaroth` does not only consider stencils in isolation, but also their interplay with other operations.
> JP comment in source below (bibliography additions)
<!--
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

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to consider technical implementation details.
The main operations, like stencils, are written in a declarative syntax, and the kernels that use them are written in an imperative syntax.
Thus, their implementation is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. 
In addition to stencils the DSL supports two other operations: 1) reductions -- which are commonly needed for stencil-based solvers and require multiple steps to perform across multiple GPUs, and 2) distibuted ray-tracing along integer coordinate lines -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].

> OL: removed some old comments here between Johannes and Touko, to summarize: "imperative" and "declarative" might be considered jargon. I think this is a case of "Google is your friend" for readers, but would be ok with a footnote.
> Please yell at me if you disagree.

Abstracting away these distributed GPU-application optimizations reduces the complexity at the DSL source level.
`acc` transpiles the DSL source into CUDA or HIP source, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread group sizes for kernel execution.

Certain changes to the run-time configuration may change the branches taken in a particular kernel.
Not knowing which branches are taken severely limits `Astaroth`'s ability to optimize the code.
Therefore `Astaroth` also supports run-time compilation, which is used to eliminate unused code and variables.

> OL: removed some previous comments, leaving some by Touko and Johannes for context

> JP: unclear how run-time compilation and branches are related. We do the CPU compilation first and assume all ifs are true. Is there some other mechanism that triggers recompilation during the actual simulation?

> TP: We do run-time compilation exactly so we do not have to assume all ifs are true (this is hardly the case for large simulation codes). Also the CPU analysis code is recompiled so it can known which branches to take. Recompilation is triggered explicitly by the user by an API function.
Modified so the text explains this more directly.

> TP: Is the meaning of what is a configuration variable the clearest? That's why I took the word configuration earlier away but it is clear that a variable on its own is too vague. Any ideas what would be a more apt naming?

>OL: what is the "execution information"

>TP: By execution information I mean information gathered by executing a specialized CPU analysis version of the code
>This information is used quite extensively for any possible purposes: stencils,reductions,ray-tracing,communication,optimizing variables out etc..
>Tried to reword the paragraph to capture this more general fact

## Multi-GPU runtime API

In the DSL, users can define an iteration of their program using a `ComputeSteps` declaration.
The `ComputeSteps` declaration consists of a list of steps, specifying which kernels to run, and which boundary conditions to impose. 
`Astaroth`'s runtime constructs a directed acyclic graph (DAG) of the steps in an iteration, where the steps are decomposed into tasks with fine-grained dependency relations.
The decomposition into tasks is based on the overall domain decomposition (necessary in a multi-GPU context), and the stencils' data access patterns.
Optimizations are also applied at this stage: kernels may be fused together if the stencils they access a common set of stencils, and a minimal communication pattern is deduced.
`Astaroth`'s task scheduler can then iterate the `ComputeSteps` any number of times, asynchronously launching tasks as prerequisite tasks are completed.
This improves performance in communication-bound cases, especially for higher process counts [@lappi2021task].
For fast data transfers and to support all possible hardware, both GPU-to-GPU remote direct memory access (RDMA) and CPU-to-CPU communication are supported.

> TP:By optimizations in the ComputeSteps I mean operations like optimizations kernel fusion and minimized amount of communication that go beyondthe TaskGraphs as they are. It would be nice to incorporate the existence of these into the text, but given we are limited for space maybe there simply is not enough space.

> OL: Added a sentence about the optimizations, is that good or is it too vague? The only thing I'm worried about is the communication pattern minimization. As a reader I would be unsure of why this is necessary. Why would we generate unnecessary commnunication tasks?

This runtime system can be accessed through `Astaroth`s runtime API.
The API is C-ABI compatible, supporting foreign function interfaces in external applications written in any programming language.
The API is organized into two layers: the `Device` layer and the `Grid` layer.
The `Device` layer provides access to single-GPU functionality, such as moving data between CPU and GPU, launching kernels, and loading/storing snapshots from disk.
The `Grid` layer provides access to multi-GPU functionality, such as executing DAGs, distributed initialization, and distributed loading/storing of snapshots.
Other special functionality is also provided through the API, such as distributed Fourier transforms.

## Solver

`Astaroth` also includes a standalone finite-difference solver, which can be used to write new simulations and works as a simple test case for performance research.
This solver has been written mostly with astrophysical MHD in mind, using the DSL code in `acc-runtime/samples/mhd_modular` as a base.
The solver can, however be modified to solve any similar problem by modifying the DSL source.

The solver takes care of distributed initial conditions, domain decomposition, simulation diagnostics, and logging.
The solver can also be configured for run-time compilation, and to periodically write out snapshots or slices of the data cube.
It is also built to react to a number of events, such as NaNs, simulation time limits, and a stop signal given through the file system.

>OL: TODO: write about MHD, TFM, features

 > OL: is this the `standalone_mpi` solver? I read through the source, and it only supports four hard coded simulation cases: MHD, shock, hydro_heatduct, and bound_test. If this is what is meant by the standalone solver, I think a better case is madwe by focusing either on the MHD solver specifically, or mention the PCA work.

 > TP: yes this is the standalone solver. The source code is horribly out of date but it is not as limited as the source code makes it out to be (the PhysicsSimulation Enums do not need to be used). And can be relatively easily extended to cover more cases, which exactly what we are doing with the Taiwanese.

> TP: Oskar, if you include something about TFM you should refer the reader to samples/tfm-mpi where the TFM solver is.

> JP comment in source below
<!--
%JP: Suggest more focus on the modular/API nature of Astaroth. Something in line (stream-of-consciousness draft follows, feel free to refine) "Domain-specialized modules can be developed using the Astaroth DSL and API. We provide MHD, TFM, sink particle, ray tracing, whatnot, modules out of the box as production-ready solvers. Third-party modules for acoustic/earthquake simulations have also been developed[@ladino_or_what_was_it_again]. Furthermore, the library has been integrated as a GPU backend for Pencil Code [@some_PC-A_reference?], expanding the the use cases further to (what?)."
%
%JP: Additional justification (can leave this out, just for information): I strongly recommend listing TFM as one of the highlights. It is production-ready for extracting turbulent transport coefficient for mean-field solar dynamo models and to my knowledge, the fastest in the world right now by a factor of $10\times$ (compared to PC[@pekkila_graphicsprocessors_2026;@schrinner_around_2006;@brandenburg_scale_dependence_2008;@warnecke_nature_around_2020], not aware of any other implementations).
%TP: agree on these but to me it sounds like they would best go to the solver. 
     and based on my experience with PC I would not overstate of the modularity of Astaroth as a solver: IMO PC is more modular and compated to it Astraroth does not seem that modular.
-->

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. Additionally it has been used in different methods papers on performance optimizations[@pekkila_graphicsprocessors_2026;@pekkila2025stencil;@pekkila2017methods], communication techniques [@pekkila2022scalable;@lappi2021task], compiler techniques[@pekkila_masters_2019;@puro2023programmatic] and on other topics [@yokelson2024soma; @puro2025gpu].
The acceleration of `Pencil Code`  is expected to increase the number of people relying on `Astaroth` as the core execution engine. The associated performance increase of 20-60x will enable more realistic astrophysical simulations in a wide range of use cases from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

> JP comment in source below
<!--
%JP: Suggest clarifying, e.g., something like (stream of consciousness, please revise) "The Astaroth framework has been used for several publications focusing on various aspects of performance optimization[@pekkila_graphicsprocessors_2026], communication techniques[@vaisala_interaction_2021;@lappi_masters;@pekkila_scalablecommunication_2022;@pekkila_graphicsprocessors_2026], compiler techniques[@pekkila_masters_2019;@pekkila_graphicsprocessors_2026;@puro_masters?], astrophysical plasma simulations[@vaisala_interaction_2021;@pekkila_graphicsprocessors_2026], gravitational waves[@roper2020numerical], seismic modeling[@ladino], and list everything else that comes to mind[@other;@references]."
%JP: "At it's current state, Astaroth provides a production-ready toolkit (API+DSL+toolbox for reductions, IO, etc) for implementing various computational physics simulations based on structured grids. By providing efficient performance and weak scaling, astaroth enables extremely high-resolution simulations on exascale systems and further research in computational astrophysics, etc, etc, what is already mentioned at the end of the current version".
-->
> TP: answer to Johannes' suggestions. In short I would not try to sell the features of Astaroth here again but simply make sure it is visible that Astaroth has been used, is used and will be used for example by the PC community

> JP comment in source below
<!--
%JP: also worth noting here or the solver section: 10x speedup in TFM performance compared to PC and 93% weak scaling to 4096 MI250X GCDs[@pekkila_graphicsprocessors_2026].
-->


# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution, who include Jörn Warnecke, Frederick Gent, Indrani Das and Ruben Krasnopolsky.

Would Maarit know the best which funding sources to cite for the development of Astaroth??


> JP comment in source below
<!--
%JP: here's what I've listed for my papers (not sure if ReSoLVE is still relevant, Maarit will know this. Likely yes because IIRC it was the funding body before ERC)
This project has received funding from the Academy of Finland, ReSoLVE Centre of Excellence, Grant/Award Number: 307411;
The European Research Council, the European Union's Horizon 2020 research and innovation program, project UniSDyn, Grant/Award Number: 818665; KAUTE Foundation, Grant/Award Numbers: 20240173 and 20250154.
We acknowledge the computational resources provided by CSC — IT Center for Science, the Aalto Science-IT project, and resources from LUMI-G through the Euro-HPC join undertaking.

%JP: Could also add Frontier computer resource acknowledgement
-->

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém, Tzu-Chun Hsu and Jack Hsu.
