---
title: 'Astaroth: A scientific computing framework for accelerating stencil computations.'
tags:
  - C/C++
  - scientific computing
  - high performance computing
  - astrophysics

authors:
  - name: Touko Puro
    orcid: 0009-0008-8632-0385
    corresponding: true
    affiliation: 1 
  - name: Johannes Pekkilä
    orcid: 0000-0002-1974-7150
    affiliation: 1 
  - name: Miikka Väisälä
    orcid: 0000-0002-8782-4664
    affiliation: 2
  - name: Oskar Lappi 
    orcid: 0000-0003-3182-8161
    affiliation: 3
  - name: Matthias Rheinhardt
    orcid: 0000-0001-9840-5986
    affiliation: 1
  - name: Hsien Shang
    orcid: 0000-0001-8385-9838
    affiliation: 4
  - name: Maarit Korpi-Lagg 
    orcid: 0000-0002-9614-2200
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

> JP:"difficult" -> can also run large simulations with CPUs. Suggest something like "the leap in parallel throughput is needed to reach state-of-the-art accuracy in computational physics simulations"

> TP: I would agree with Oskar's wording that difficult is an apt phrasing. GPUs are never strictly needed, but clearly it is easier to run larger simulations with them.

To address this, `Astaroth` is a GPU framework for stencil computations, that has been developed with scalable scientific computing in mind.

`Astaroth` provides its own domain specific language (DSL), in which researchers can express their required computations without having to focus on technical implementation details.
It can run efficiently both on CUDA- and HIP-based environments --- and even on hardware lacking GPUs, e.g. for testing purposes.
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing and has library integrations for performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
To further ease the development and acceleration of PDE solvers based on the finite-difference method, `Astaroth` comes together with its own PDE solver and a standard library with support for a variety of PDE-solver functionality.
`Astaroth` has primarily been used for turbulent astrophysical plasma simulations.

# Statement of need

Much of the software used for scientific computing is written for CPUs, and has to be ported to GPUs to run larger problems with decent times-to-solution.
`Astaroth` has been developed to solve this problem for the subset of scientific software that relies on stencil computations.
`Astaroth`'s DSL can be used to rewrite existing PDE solvers or to write completely new ones.
As an example, `Astaroth` has been used to scale an astrophysical plasma simulation to thousands of GPUs with a weak scaling efficiency \>90% [@pekkila_graphicsprocessors_2026]. 

Accelerating such simulations was the original reason for the creation of `Astaroth`.
A widely used framework for them is the Pencil Code [@brandenburg2020pencil], which is a modular multiphysics PDE solver.
The early stages of `Astaroth` development focused on implementing the high-order stencil methods employed by Pencil Code for isothermal hydrodynamics [@pekkila2017methods;@vaisala_magneticphenomena_2017]. 
With later revisions `Astaroth` has successfully been used to accelerate Pencil Code [@puro2023programmatic] with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and convolutions, are traditionally expressed using stencils.
`Astaroth` enables this task by cleanly separating the front-end (DSL) from the back-end (compiler and runtime system), which also provides researchers with a platform for performance research.

> OL: suggest moving the bracketed phrase to the end of the sentence for better flow, also suggest reordering the three points about stencil computations, DSL, and scalability to make a stronger argument: Problem -> Solution -> Result.

> BEGIN edit proposal (no changes to the first two sentences here)

> Much of the software used for scientific computing is written for CPUs, and has to be ported to GPUs to run larger problems with decent times-to-solution.
> `Astaroth` has been developed to solve this problem for the subset of scientific software that relies on stencil computations.
> `Astaroth`'s DSL can be used to rewrite existing PDE solvers or to write completely new ones.

> END edit proposal


> JP: **scales**

> - better to use the thesis (2022 paper only goes up to 64 devices, in the thesis we do 4096).

> - "scales" -> has been demonstrated to scale weakly at ?% efficiency to 64 devices in MHD simulations (2022 paper, could leave this out and focus on the TFM case). Has been demonstrated to scale weakly at 93% efficiency to 4096 devices in computations with the test-field method. 

>TP: Made the statement now more precise by having the weak scaling efficiency in brackets. What do you think does that suffice? If the reader cares about details they can lookup Johannes' thesis.

> JP: some notes if we're going to include history and our prior work

> Astaroth Code: hard-coded compressible hydrodynamics 2014-2019[@vaisala_magneticphenomena_2017;@pekkila2017methods]

> Astaroth: generalized stencil framework 2019- (DSL V1, single-gpu) [@pekkila_masters_2019], rewritten DSL V2 grammar and code generator [@pekkila_stencilcomputations_2025]

> Astaroth: single-node multi-gpu[@vaisala_interactionlarge_2021], general MPI halo exchange communication and Z-order [@pekkila2022scalable], distributed reductions(lappi, not sure if mentioned in the thesis) and task system[@lappi], topology-aware domain decomposition and mapping and communication optimizations (fused packing) and TFM[@pekkila_graphicsprocessors_2026]

> Astaroth: CPU, PC-A, DSL improvements, ray tracing, and other contributions [@Touko'sWork]

> MV: I have edited this to be able to properly include my thesis work as the context within the codes development history. 

# State of the field                                                                                                                  
> JP revised suggestion START

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

> JP revised suggestion END

> JP: commented out previous version (this below)
> Methods to accelerate and improve perfomance-portability and productivity of stencil computations are widely studied. We refer the reader to [pekkila_graphicsprocessors_2026] for more details on the background.
> There are widely used lower-level tools such as OpenMP[dagum1998openmp], OpenACC[openacc2025spec], Kokkos[@trott2021kokkos] and Raja[@beckingsale2019raja] which provide abstraction layers for parallel computational patterns, but which still leave e.g. performing the required communications to the user. 
> Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[mullapudi2015polymage]. Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
> Closest to Astaroth are domain-specialized frameworks providing the building blocks for computational physics, such as Cactus[@goodale_cactusframework_2003] and Parthenon[@grete_parthenonperformance_2023].
> Compared to these projects, `Astaroth` uses a DSL, resembling the approaches of Halide, Polymage, and Patus for their domains.
> Distinctively to existing approaches, `Astaroth` specializes, and has extensive optimizations, for cache-constrained use cases, such as multiphysics simulations which need many interdepent values in working memory at the same time. Additionally, `Astaroth` incorporates other needed operations, like reductions, together with stencils to cover use cases that are not fully stencil-based.


> JP START: added bullet points of things

> - methods to accelerate and improve perfomance-portability and productivity Stencil computations widely studied: numerous software projects at different levels of abstractions. We refer the reader to [pekkila_graphicsprocessors_2026] for more details on the background.

> - Low-level generalized approaches suitable for implementing stencil solvers for graphics processors include CUDA and HIP [@nvidia_cudac_2025;@amd_hipdocumentation_2024] (computation) and GPU-aware MPI[messagepassinginterfaceforum_openmpi_2025;argonnenationallaboratoryandmississippistateuniversity_mpich_2026] (communication).

> - OpenMP[dagum1998openmp], OpenACC[openacc2025spec], Kokkos[@trott2021kokkos], Raja[@beckingsale2019raja] provide abstractions and portability layers for parallel computational patterns and are suitable for implementing domain-specialized solvers on shared-memory systems.

> - Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[mullapudi2015polymage]. Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].

> - Closest to Astaroth are domain-specialized frameworks providing the building blocks for computational physics, such as Cactus[@goodale_cactusframework_2003] and Parthenon[@grete_parthenonperformance_2023].

> - Closest to our work is the Parthenon, which provides a framework for distributed computations of  adaptive mesh refinement, utilizing Kokkos as the backend for shared-memory computation.

> - Like these projects, Astaroth provides ready-made components for implementing computation and communication for their domain-specialized case.

> - Compared to these projects, Astaroth uses a domain-specific language that is lowered into platform-portable autotuned CUDA/HIP code to implement efficient stencil computations, resembling the approaches of Halide, Polymage, and Patus for their domains.

> - The distinctive feature of Astaroth is that it specializes in cache-constrained computations on structured grids especially in multiphysics simulations, where the coupling of physical fields, the use of large stencils, and the use of double-precision arithmetic. These requirements result in prohibitively large working sets in on-chip memory for obtaining sufficient levels of reuse to break through the memory bandwidth bound. Astaroth implements a code generator for unrolling and reordering the stencil computations to reduce instruction counts, improve the locality of memory accesses, and ensuring sufficient instruction-level parallelism to obtain improved latency-hiding at low occupancy due to high register allocation per thread to improve the reuse of intermediate values.


>TP: Made now a draft of the body text based on Johannes' bullet points. Johannes, thoughts? Please edit if you don't like it.

>JP: OK. Revised above (the uncommented one).


# Software design

`Astaroth` consists of three main components: 1) `acc`, a compiler and runtime for a domain-specific language (DSL) for stencil computations, 2) an API for executing stencil applications on multi-GPU platforms, and 3) a standalone solver for certain simulation cases.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## `acc` compiler and runtime

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to consider technical implementation details.
The main operations, like stencils, are written in a declarative syntax, and the kernels that use them are written in an imperative syntax.
The implementation of the operators is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. 
Of central importance of these is the unrolled computation of all required stencils at the start of the kernels, which enables instruction-level parallelism and efficient usage of software-managed caches [@pekkila_graphicsprocessors_2026].
In addition to stencils, the DSL supports two other operations: 1) reductions -- which are commonly needed for stencil-based solvers and require multiple steps to perform across multiple GPUs, and 2) distributed simplified ray-tracing, where rays are restricted to move through neighbouring grid points, -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].
Additionally, `Astaroth` comes with its own standard library for the DSL, which, in addition to other functionality, provides derivative operators needed for PDE solvers, which are implemented for generally spaced Cartesian,spherical and cylindrical grids, and Poisson solvers needed for e.g. self-gravity.

> OL: imperative and declarative, TODO: write a footnote

`acc` transpiles the DSL source into CUDA or HIP source code, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread block sizes for kernel execution.
Certain changes to the run-time configuration may change the branches taken in a particular kernel.
Not knowing which branches are taken severely limits `Astaroth`'s ability to optimize the code.
Therefore `Astaroth` also supports run-time compilation, which is used to eliminate unused code and variables.


> OL: RE: branching. There was some confusion about this. Maybe it is better to talk directly about conditional compilation, as that is what is being discussed here. The following edit suggestion would replace the last three sentences.

> OL: What do you think? I know I'm the "dictator" of this section, but would like some consensus, since we've already discussed this part of the text quite a bit.

> OL: BEGIN EDIT SUGGESTION

> `Astaroth` also supports conditional compilation, only compiling code related to those kernels that are actually executed at run-time.
> This can also be done based on config variables, which may affect conditional statements in the code, through run-time compilation.
> This does require an extra compilation step at the start of run-time, but is a significant performance optimization, and is recommended especially when running very large problem sizes.

> OL: END EDIT SUGGESTION


## Multi-GPU runtime API

In the DSL, users can define a list of compute steps specifying which kernels to run and which boundary conditions to impose.
`Astaroth`'s multi-GPU runtime constructs a directed acyclic graph (DAG) of the compute steps, where each step is decomposed into computation and communication tasks.
The decomposition into tasks is based on the overall domain decomposition, and the stencils' data access patterns, which also determines dependency relations between the tasks.
As an optimization, kernels may be fused together to reduce memory reads.

> TP: A list of `ComputeSteps` is not meaningful, sorry if I was unclear about this. Would suggest the edit: "In the DSL, user can declare lists of steps, specifying which kernels to run, and which boundary conditions to impose, which are called `ComputeSteps`.
> TP: ... (DAG) of the steps in an iteration... ", would drop the word iteration since it makes the text harder to understand and as discussed `ComputeSteps` can be something you invoke only once. IMO it would be more clear Steps ---> DAG and the iteration point of view comes from the when the scheduler executes it, as you have no written later.

> OL: agree that it is not meaningful. Removed references to iterations, and focused more on ComputeSteps as the mental model of the user.
> TP: The wording is still not totally correct. ComputeSteps refers to the whole list, and now the wording means the user would specify a list of these lists. Maybe something like: "define a list of steps, ..., which are called `ComputeSteps`.
> OL: I see, you see the `ComputeSteps` is a proper noun for the language feature. I have now replaced it with a natural language concept, and left out the keyword from the paper.
> As the paper itself is not in-depth documentation, I think this is enough, as users can look up DSL syntax/keywords in the docs.

`Astaroth`'s task scheduler executes these DAGs, asynchronously launching computation and communication tasks as prerequisite tasks are completed.
This improves performance in communication-bound cases, especially for higher process counts [@lappi2021task].
For fast data transfers and to support all possible hardware, both GPU-to-GPU remote direct memory access (RDMA) and CPU-to-CPU communication are supported.

This runtime system can be accessed through `Astaroth`s runtime API.
The API is C-ABI compatible, supporting foreign function interfaces to external applications written in any programming language.
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
It is also built to react to a number of events, such as NaNs in the simulation data, simulation time limits, and a stop signal given through the file system.

> TP: suggested edit (again if you agree with the edit do it and remove this, if not keep this comment): would not think it is that important to explicitly mention the support for run-time compilation of the standalone-solver: any Astaroth based code can easily ,not trivially but still easily, be run with run-time compilation so would not stress this. Also maybe my brain is stuck on the solvers I know, but would 100% assume any solver periodically writes out snapshots or slices without the text explicit mentioning this fact. So in summary: would simply remove the second sentence all together. The reactivity is certainly nice to mention since most codes I know are not as clean in this regard as the standalone solver.

> OL: TODO: write about MHD, TFM, features

> OL: is this the `standalone_mpi` solver? I read through the source, and it only supports four hard coded simulation cases: MHD, shock, hydro_heatduct, and bound_test. If this is what is meant by the standalone solver, I think a better case is madwe by focusing either on the MHD solver specifically, or mention the PCA work.

> TP: yes this is the standalone solver. The source code is horribly out of date but it is not as limited as the source code makes it out to be (the PhysicsSimulation Enums do not need to be used). And can be relatively easily extended to cover more cases, which exactly what we are doing with the Taiwanese.

> TP: Oskar, if you include something about TFM you should refer the reader to samples/tfm-mpi where the TFM solver is.

> JP: Suggest more focus on the modular/API nature of Astaroth. Something in line (stream-of-consciousness draft follows, feel free to refine) "Domain-specialized modules can be developed using the Astaroth DSL and API. We provide MHD, TFM, sink particle, ray tracing, whatnot, modules out of the box as production-ready solvers. Third-party modules for acoustic/earthquake simulations have also been developed[@ladino_or_what_was_it_again]. Furthermore, the library has been integrated as a GPU backend for Pencil Code [@some_PC-A_reference?], expanding the the use cases further to (what?)."

>JP: Additional justification (can leave this out, just for information): I strongly recommend listing TFM as one of the highlights. It is production-ready for extracting turbulent transport coefficient for mean-field solar dynamo models and to my knowledge, the fastest in the world right now by a factor of $10\times$ (compared to PC[@pekkila_graphicsprocessors_2026;@schrinner_around_2006;@brandenburg_scale_dependence_2008;@warnecke_nature_around_2020], not aware of any other implementations).

>TP: agree on these but to me it sounds like they would best go to the solver.
> and based on my experience with PC I would not overstate of the modularity of Astaroth as a solver: IMO PC is more modular and compared to it Astraroths solver does not seem that modular.
> MR: if there is no better place, this isection should say someth about the built-in entities (operators, grid-related stuff etc.)

# Research impact statement

> MV: I moved the mention of my PhD thesis to earlier part of the text. This is because at the thesis I just present the first attempt at Astaroth. At the time of my thesis we were not yet able to do physics with it beyond simple tests. [@vaisala2021interaction] was the first real paper to present Astaroth based physics results. 

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. 
Additionally it has been used for research on performance optimization methods[@pekkila_graphicsprocessors_2026;@pekkila2025stencil;@pekkila2017methods], communication techniques [@pekkila2022scalable;@lappi2021task], compiler techniques[@pekkila_masters_2019;@puro2023programmatic] and other topics [@yokelson2024soma; @puro2025gpu].
The acceleration of `Pencil Code`  is expected to increase the number of people relying on `Astaroth` as the core execution engine. The associated performance increase of 20-60x will enable more realistic astrophysical simulations in a wide range of use cases from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

> JP: Suggest clarifying, e.g., something like (stream of consciousness, please revise) "The Astaroth framework has been used for several publications focusing on various aspects of performance optimization[@pekkila_graphicsprocessors_2026], communication techniques[@vaisala_interaction_2021;@lappi_masters;@pekkila_scalablecommunication_2022;@pekkila_graphicsprocessors_2026], compiler techniques[@pekkila_masters_2019;@pekkila_graphicsprocessors_2026;@puro_masters?], astrophysical plasma simulations[@vaisala_interaction_2021;@pekkila_graphicsprocessors_2026], gravitational waves[@roper2020numerical], seismic modeling[@ladino], and list everything else that comes to mind[@other;@references]."

> JP: "At it's current state, Astaroth provides a production-ready toolkit (API+DSL+toolbox for reductions, IO, etc) for implementing various computational physics simulations based on structured grids. By providing efficient performance and weak scaling, astaroth enables extremely high-resolution simulations on exascale systems and further research in computational astrophysics, etc, etc, what is already mentioned at the end of the current version".

> TP: answer to Johannes' suggestions. In short I would not try to sell the features of Astaroth here again but simply make sure it is visible that Astaroth has been used, is used and will be used for example by the PC community

> JP: also worth noting here or the solver section: 10x speedup in TFM performance compared to PC and 93% weak scaling to 4096 MI250X GCDs[@pekkila_graphicsprocessors_2026].

> OL: I agree with Johannes about more references, but don't really care which way they are listed. Either way is fine, although I think the gravitational waves paper was PC not Astaroth.

> TP: I have tried to give all references that now come to mind, including those in Johannes' suggestion. If you spot one is missing please simply add it. Yes, the gravitational waves paper is for PC and to showcase what will be done in the future with PC.

# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution. These include Jörn Warnecke, Frederick Gent, Indrani Das and Ruben Krasnopolsky.

> TP: Would Maarit know the best which funding sources to cite for the development of Astaroth??

> MV: Good question! What is the policy here? How far we should go? Like should I mention Wihuri and SKR because I got funding from then for my PhD?

> JP: here's what I've listed for my papers (not sure if ReSoLVE is still relevant, Maarit will know this. Likely yes because IIRC it was the funding body before ERC)
> This project has received funding from the Academy of Finland, ReSoLVE Centre of Excellence, Grant/Award Number: 307411;
> The European Research Council, the European Union's Horizon 2020 research and innovation program, project UniSDyn, Grant/Award Number: 818665; KAUTE Foundation, Grant/Award Numbers: 20240173 and 20250154.
> We acknowledge the computational resources provided by CSC — IT Center for Science, the Aalto Science-IT project, and resources from LUMI-G through the Euro-HPC joint undertaking.

> JP: Could also add Frontier computer resource acknowledgement

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém, Tzu-Chun Hsu and Jack Hsu.
