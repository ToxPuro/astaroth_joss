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
`Astaroth` is a GPU framework for stencil computations, that has been developed to address this problem of scalable scientific computing.
`Astaroth` provides its own domain specific language (DSL), in which researchers can express their required computations without having to focus on technical implementation details.
It can run efficiently both on CUDA- and HIP-based environments --- and even on hardware lacking GPUs, e.g. for testing purposes.
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing and has library integrations for performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
`Astaroth` has primarily been used for turbulent astrophysical plasma simulations.

# Statement of need

Much of the software used for scientific computing is written for CPUs, and has
to be ported to GPUs to run larger problems with decent times-to-solution.
`Astaroth` has been developed to solve this problem for the subset of
scientific software that relies heavily on stencil computations.
`Astaroth`'s DSL can be used to rewrite existing PDE solvers or to write
completely new ones.
As an example, `Astaroth` has been used to write a PDE solver for astrophysical
plasma simulations, which scales to thousands of GPUs with a weak scaling
efficiency \>90% [@pekkila_graphicsprocessors_2026]. 

Accelerating such simulations was the original reason for the creation of
`Astaroth`.
A widely used framework for them is the Pencil Code [@brandenburg2020pencil],
which is a modular multiphysics PDE solver.
The early stages of `Astaroth` development focused on implementing the
high-order stencil methods employed by Pencil Code for isothermal hydrodynamics
[@pekkila2017methods;@vaisala_magneticphenomena_2017]. 
With later revisions `Astaroth` has successfully been used to accelerate Pencil
Code [@puro2023programmatic] with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither
is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and
convolutions, are traditionally expressed using stencils.


> TP: reworded the text about the scalability to make the text go together with the mentioning of Astaroth's PDE solver, which was not clearly defined before.

> MV: I have edited this to be able to properly include my thesis work as the context within the codes development history. 
> OL: cleaned up comments on scalability. Is this discussion below about history resolved? Most of Johannes' suggestions are not in the text.
> TP: Johannes and Miikka are you now happy with the history point of view? I would be. I understood Johannes' notes to simply to something to use for writing: we don't have the space to document the history of the code in real detail and it is not IMO interesting to the reader.
> MV: I think the general content is ok. Excuse me for making the text better readable for a text editor. I do not know what editor you guys are using, but reading the document on Neovim is horrible because of how the text is laid out.


# State of the field                                                                                                                  

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

> TP: We have tried to remove words we can not defend or quantify like "ergonomic" and "compact". I would say now the word "modular" is one such word again. What do you think? I would be fine also with dropping this sentence since also we should not advertise different physics cases Astaroth comes since IMO they are not modular or expansive enough to advertise.

A distinctive feature of Astaroth is its specialization for cache-constrained use cases, especially in multiphysics simulations where interdependent values need to be held in working memory at the same time. Additionally, `Astaroth` does not only consider stencils in isolation, but also their interplay with other operations.





# Software design

`Astaroth` consists of three main components: 1) `acc`, a compiler and runtime for a domain-specific language (DSL) for stencil computations, 2) an API for executing stencil applications on multi-GPU platforms, and 3) a standalone solver for certain simulation cases.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## `acc` compiler and runtime

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to consider technical implementation details.
The main operations, like stencils, are written in a declarative syntax, and the kernels that use them are written in an imperative syntax. [^paradigm_footnote]
The implementation of the operators is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. 
Of central importance of these is the unrolled computation of all required stencils at the start of the kernels, which enables instruction-level parallelism and efficient usage of software-managed caches [@pekkila_graphicsprocessors_2026].
In addition to stencils, the DSL supports two other operations: 1) reductions -- which are commonly needed for stencil-based solvers and require multiple steps to perform across multiple GPUs, and 2) distributed simplified ray-tracing, where rays are restricted to move through neighbouring grid points, -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].
Additionally, `Astaroth` comes with its own standard library for the DSL, which, in addition to other functionality, provides derivative operators needed for PDE solvers, which are implemented for generally spaced Cartesian,spherical and cylindrical grids, and Poisson solvers needed for e.g. self-gravity.

`acc` transpiles the DSL source into CUDA or HIP source code, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread block sizes for kernel execution.
`Astaroth` also supports conditional compilation based on the conditional branches in the DSL source[^branch_footnote] --- only compiling code that is actually executed at run-time.
Because configuration variables may affect conditional branches taken in the code, `Astaroth` also supports run-time compilation.
This does require an extra compilation step at the start of run-time, but is a significant performance optimization, and is recommended especially when running very large problem sizes.

> OL: RE: branching. There was some confusion about this. Maybe it is better to talk directly about conditional compilation, as that is what is being discussed here. I've replaced the paragraph in the source. The old one is below.
> OL: RE: conditional compilation is confusing:  What we're doing is conditional compilation. The preprocessor #if, a C++ if constexpr, and the if/switch statements in Astaroth are all examples of branches that are eliminated through conditional compilation. The flags given to the build system are the "values that change" in the preprocessor example, in the Astaroth example the "values that change" are the config values. But they are the same procedure, just with different syntax and interfaces.

> OL: RE: the suggested sentence is better, but it is still very verbose. I took some of the suggestions and added them to the new paragraph above. I also tried to rework in more clearly that we're dealing with branches, now using the full "conditional branch" term. I also added a footnote to explain branching to readers who are unfamiliar.

> TP: Edit suggestion: "...based on the conditional branches in the DSL source ..." --> "based on the conditional statements in the DSL source ...", conditional statement should be clear for every reader and we skip the need for a footnote.

> TP: In short: I believe for the user POV (and at least from my POV) preprocessor statements, build flags and so on are quite different entities than say the amount of rotation of the coordinate frame omega, which would be one of the config variables dictating control-flow. But we luckily do not have to give our viewpoints, so this is not important.

> TP: On conditional compilation: Sorry it took for me so long to see your point but your point was that you want to clearly showcases these orthogonal features of conditional and run-time compilation, I get it now. That is actually quite nice in the text! Sorry again for being a bit stubborn, but the final text is now nice! 

> TP: We should still improve the last sentence: runtime-compilation happens not only at the start of the run-time and as I stated earlier it is needed regardless of the size of the setup if the user would like to use for example `stdlib/general_operators.h`, now the text would give the wrong idea that one can not use it for small setups, but it is more about what features they want to use and depending on what very large is understood to mean I would not say the setup has to be very large. In the end is this sentence giving any new information to the reader: runtime-compilation already means extra processing has to happen at run-time and it is an optimization and those are naturally more important at large sizes and as said the optimization viewpoint is not totally accurate.

> TP: At the cost of nice conciseness and directeness (the text is indeed very nice) we have lost some information on how execution information is used for other optimizations than code elimination like in tasks and for memory allocations and communication, which are not about code size. Would be nice if we could get that viewpoint back to the text. Maybe replacing the last sentence with something like. "These together allow `Astaroth` to know what code will get executed and to use this for further optimizations  when e.g. allocating memory and forming communication patterns."

> OL: BEGIN OLD VERSION 

> `acc` transpiles the DSL source into CUDA or HIP source code, which is further compiled into machine code using a native CUDA or HIP compiler.
> The program thus produced is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread block sizes for kernel execution.
> Certain changes to the run-time configuration may change the branches taken in a particular kernel.
> Not knowing which branches are taken severely limits `Astaroth`'s ability to optimize the code.
> Therefore `Astaroth` also supports run-time compilation, which is used to eliminate unused code and variables.

> OL: END OLD VERSION


## Multi-GPU runtime API

In the DSL, users can define a list of compute steps specifying which kernels to run and which boundary conditions to impose.
`Astaroth`'s multi-GPU runtime constructs a directed acyclic graph (DAG) of the compute steps, where each step is decomposed into computation and communication tasks.
The decomposition into tasks is based on the overall domain decomposition, and the stencils' data access patterns, which also determines dependency relations between the tasks.
As an optimization, kernels may be fused together to reduce memory reads.

> TP: What do you think is it worth mentioning that the system drops unnecessary calls i.e. those without observable effect due to configuration variables? If yes, then I would propose the following: "As an optimization, unecessary calls are dropped and kernels may be fused together to reduce memory reads."
> OL: that can be mentioned if there are words left in the budget. But a "call" is ambiguous". A call to what? A kernel? Needs to be specified.

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

`Astaroth` also includes a standalone finite-difference solver, which takes full advantage of the DSL and runtime API, and can be used to write new simulations.
It also works as a testbed for performance research.
This solver uses an astrophysical magnetohydrodynamical setup (`acc-runtime/samples/mhd_modular`) by default, but can be configured to run any DSL code.
The samples directory also includes other production-ready setups, e.g. `tfm-mpi` for the test-field method [@johannes_paper].

>TP: Changed MHD ---> magnetohydrodynamical since we have not defined MHD. And example --- > setup. Example IMO invokes that it is only for e.g. learning.
>TP: samples ---> setups, IMO setup is easier to grasp for outside reader than sample. 

The solver takes care of distributed initial conditions, domain decomposition, simulation diagnostics, and logging.
It is also built to react to a number of events, such as NaNs in the simulation data, simulation time limits, and a stop signal given through the file system.
`analysis/` contains Python-based data analysis tools, which can be used to process and work with the data produced by the standalone solver. 

> JP: Suggest more focus on the modular/API nature of Astaroth. Something in line (stream-of-consciousness draft follows, feel free to refine) "Domain-specialized modules can be developed using the Astaroth DSL and API. We provide MHD, TFM, sink particle, ray tracing, whatnot, modules out of the box as production-ready solvers. Third-party modules for acoustic/earthquake simulations have also been developed[@ladino_or_what_was_it_again]. Furthermore, the library has been integrated as a GPU backend for Pencil Code [@some_PC-A_reference?], expanding the the use cases further to (what?)."

>JP: Additional justification (can leave this out, just for information): I strongly recommend listing TFM as one of the highlights. It is production-ready for extracting turbulent transport coefficient for mean-field solar dynamo models and to my knowledge, the fastest in the world right now by a factor of $10\times$ (compared to PC[@pekkila_graphicsprocessors_2026;@schrinner_around_2006;@brandenburg_scale_dependence_2008;@warnecke_nature_around_2020], not aware of any other implementations).

>TP: agree on these but to me it sounds like they would best go to the solver.
> and based on my experience with PC I would not overstate of the modularity of Astaroth as a solver: IMO PC is more modular and compared to it Astraroths solver does not seem that modular.

> OL: I think the test-field methods may be best expanded in research impact. I've mentioned them here for now. 

> TP: dropped the seismology module: there indeed is no such module. They have made a PR for solving the wave equation, which is a long way from a proper third-party module for earthquake simulations. It is correct that Astaroth has been used in seismology but not that there is a module for it.

> OL: ah, shame. It's not a module, but could it be argued that it is a "setup"? We could also talk about work-in-progress on a third party module and reference the paper.

# Research impact statement

> MV: I moved the mention of my PhD thesis to earlier part of the text. This is because at the thesis I just present the first attempt at Astaroth. At the time of my thesis we were not yet able to do physics with it beyond simple tests. [@vaisala2021interaction] was the first real paper to present Astaroth based physics results. 

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. 
Additionally it has been used for research on performance optimization methods[@pekkila_graphicsprocessors_2026;@pekkila2025stencil;@pekkila2017methods], communication techniques [@pekkila2022scalable;@lappi2021task], compiler techniques[@pekkila_masters_2019;@puro2023programmatic] and other topics [@yokelson2024soma; @puro2025gpu].
We expect that the acceleration of `Pencil Code` with `Astaroth` will increase the number of `Astaroth` users.
The associated performance increase of 20-60x will enable more realistic astrophysical simulations in a wide range of use cases from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

> OL: Edit suggestion added for penultimate sentence, old sentence below. "People relying on" is fuzzy, would prefer simply "users". Also used active voice to make it clear who it is that expects this migration to happen. 
> TP: Yes agree that wording was fuzzy. The reason why I did not use the word users was that I was not 100% sure what constitutes a user of Astaroth when they do not directly interact with it, or are not necessarily are aware of its existence, but maybe my worry was overblown.
> OL: Ok, so then this sentence is indeed about the PCA interface. In that case I feel we really do need to mention it. Because otherwise, if e.g. a pencil code user reads this paper, the only reference to solvers is the standalone solver, and I think that is unhelpful. It can be mentioned as work in progress.

> OL: BEGIN OLD SENTENCE

> The acceleration of `Pencil Code` with `Astaroth` is expected to increase the number of people relying on `Astaroth` as the core execution engine.

> OL: END OLD SENTENCE


> JP: Suggest clarifying, e.g., something like (stream of consciousness, please revise) "The Astaroth framework has been used for several publications focusing on various aspects of performance optimization[@pekkila_graphicsprocessors_2026], communication techniques[@vaisala_interaction_2021;@lappi_masters;@pekkila_scalablecommunication_2022;@pekkila_graphicsprocessors_2026], compiler techniques[@pekkila_masters_2019;@pekkila_graphicsprocessors_2026;@puro_masters?], astrophysical plasma simulations[@vaisala_interaction_2021;@pekkila_graphicsprocessors_2026], gravitational waves[@roper2020numerical], seismic modeling[@ladino], and list everything else that comes to mind[@other;@references]."

> JP: "At it's current state, Astaroth provides a production-ready toolkit (API+DSL+toolbox for reductions, IO, etc) for implementing various computational physics simulations based on structured grids. By providing efficient performance and weak scaling, astaroth enables extremely high-resolution simulations on exascale systems and further research in computational astrophysics, etc, etc, what is already mentioned at the end of the current version".

> TP: answer to Johannes' suggestions. In short I would not try to sell the features of Astaroth here again but simply make sure it is visible that Astaroth has been used, is used and will be used for example by the PC community

> JP: also worth noting here or the solver section: 10x speedup in TFM performance compared to PC and 93% weak scaling to 4096 MI250X GCDs[@pekkila_graphicsprocessors_2026].

> OL: I agree with Johannes about more references, but don't really care which way they are listed. Either way is fine, although I think the gravitational waves paper was PC not Astaroth.

> TP: I have tried to give all references that now come to mind, including those in Johannes' suggestion. If you spot one is missing please simply add it. Yes, the gravitational waves paper is for PC and to showcase what will be done in the future with PC.

> OL: should there be more highlights on the domain science side? Currently the only highlight is the performance

> OL: The pencil code accelleration is mentioned without explanation of the PCA setup. I read it as referring to this development, it should be expanded to explain this to the reader (not a lot of text, just one or two sentences).

> TP: Do you mean we have to repeat that PC was accelerated with PC? Now I changed "Acceleration of Pencil Code ---> Acceleration of Pencil Code with Astaroth". And if you mean that the text would read like that the acceleration of PC is what is described here I would not be sure since in this section we give anyways use cases of Astaroth so then it would simply one use case among others. What more should we say and do you think we could make the line more clear?

> OL: to me, the change that would drive users to Astaroth is the PCA transpiler method, as that allows PC users to keep their own methods. I don't think PC users will be migrating to the standalone solver. But as long as we make it clear that this is just an expectation that WE have, I guess it's fine. Made an edit suggestion about this above.

> MV: I believe that the future of standalone solver is in the Python interface. However, even though with Ondrej's thesis work will like do a lot for it to become reality, it is maybe too new development to be feature in this paper yet. What do you think? 

> TP: Yes, surely it will be important and a great advance but given that the work is just starting I would not include it yet, since the reader would get the impression that the Python interface work is further along than it is.

# Acknowledgements

We acknowledge the contributions of every committer and code contributor to Astaroth[^contributor_footnote] and the early users of it who have been instrumental in its evolution. These include Jörn Warnecke, Frederick Gent, Ruben Krasnopolsky, Wei-Wen Li and Indrani Das.

> TP: Would Maarit know the best which funding sources to cite for the development of Astaroth??

> MV: Good question! What is the policy here? How far we should go? Like should I mention Wihuri and SKR because I got funding from then for my PhD?

> TP: Would leave it to Maarit to know best which ones to mention. The JOSS paper for PC has approx ~5 sources mentioned so we should definitely not need more than that so maybe the 3 most important?

> JP: here's what I've listed for my papers (not sure if ReSoLVE is still relevant, Maarit will know this. Likely yes because IIRC it was the funding body before ERC)
> This project has received funding from the Academy of Finland, ReSoLVE Centre of Excellence, Grant/Award Number: 307411;
> The European Research Council, the European Union's Horizon 2020 research and innovation program, project UniSDyn, Grant/Award Number: 818665; KAUTE Foundation, Grant/Award Numbers: 20240173 and 20250154.
> We acknowledge the computational resources provided by CSC — IT Center for Science, the Aalto Science-IT project, and resources from LUMI-G through the Euro-HPC joint undertaking.

> JP: Could also add Frontier computer resource acknowledgement

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^paradigm_footnote]: In declarative programming, computations are defined by describing what the results look like; in imperative programming, by describing the steps to perform.
[^branch_footnote]: Conditionally branching programming constructs are if-statements and switch-statements.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém and Tzu-Chun Hsu. 

# Notes 

> JP: some notes if we're going to include history and our prior work

> Astaroth Code: hard-coded compressible hydrodynamics 2014-2019[@vaisala_magneticphenomena_2017;@pekkila2017methods]

> Astaroth: generalized stencil framework 2019- (DSL V1, single-gpu) [@pekkila_masters_2019], rewritten DSL V2 grammar and code generator [@pekkila_stencilcomputations_2025]

> Astaroth: single-node multi-gpu[@vaisala_interactionlarge_2021], general MPI halo exchange communication and Z-order [@pekkila2022scalable], distributed reductions(lappi, not sure if mentioned in the thesis) and task system[@lappi], topology-aware domain decomposition and mapping and communication optimizations (fused packing) and TFM[@pekkila_graphicsprocessors_2026]

> Astaroth: CPU, PC-A, DSL improvements, ray tracing, and other contributions [@Touko'sWork]

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

