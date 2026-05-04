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

> JP: Suggest "like" -> such as or whatever is appropriate for the context (less colloquial). Applies to all instances of "like"

> JP: Suggest oxford comma ", and integrates"


# Statement of need

Much of the software used for scientific computing is written for CPUs, and has
to be ported to GPUs to run larger problems with decent times-to-solution.
`Astaroth` has been developed to solve this problem for the subset of
scientific software that relies heavily on stencil computations.
`Astaroth`'s DSL can be used to rewrite existing PDE solvers or to write
completely new ones.
As an example, `Astaroth` has been used to write a PDE solver for astrophysical
plasma simulations [@vaisala2023exploring], which scales to thousands of GPUs with a weak scaling
efficiency \>90% [@pekkila_graphicsprocessors_2026]. 

Accelerating such simulations was the original reason for the creation of
`Astaroth`.
A widely used framework for them is the Pencil Code [@brandenburg2020pencil],
which is a modular multiphysics PDE solver.
The early stages of `Astaroth` development focused on implementing the
high-order stencil methods of Pencil Code for isothermal hydrodynamics
[@pekkila2017methods;@vaisala_magneticphenomena_2017]. 
With later revisions, `Astaroth` has successfully been used to accelerate Pencil
Code [@puro2023programmatic] with speedups of 20-60x [@pekkila2022scalable].
Of course, `Astaroth`'s PDE solver is not limited to astrophysics, and neither
is `Astaroth` limited to PDE's.
As an example, many image processing techniques, like edge detection and
convolutions, are traditionally expressed using stencils.


# State of the field                                                                                                                  

Methods to achieve performance portability in stencil computations have been widely studied.
Domain-specific languages for image processing include Halide[@ragan2013halide] and Polymage[@mullapudi2015polymage].
Autotuning code-generation frameworks include Patus[@christen_patuscode_2011] and PARTANS[@lutz_partansautotuning_2013].
More generalized software projects that provide the building blocks for domain-specialized libraries have also been proposed.
Delite[@sujeeth_delitecompiler_2014] and Lift[@steuwer_liftfunctional_2017] provide intermediate languages as targets for domain-specific languages. <!--% JP (can be left out if no room) -->
Kokkos[@trott2021kokkos] and RAJA[@beckingsale2019raja] provide abstraction layers for parallel computational patterns but focus on single-process computations.
The Chapel[@callahan_cascadehigh_2004] and Charm++[@kale_charmportable_1993] provide programming models for parallel and distributed computations.
In a more specialized approach, the Cactus framework[@goodale_cactusframework_2003] provides a collection of functionalities shared between computational science tasks.
We refer the reader to [@pekkila_graphicsprocessors_2026] for more details on the background.

> JP: "single-process computations" not sure if this is true. Would be clearer to emphasize that they focus on single-node computations (=shared memory). Also Kokkos is working on support for distributed memory (=MPI) but AFAIK it's not yet ready for production.
> TP: We had the opposite worry with Matthias that single-node performance is immediately clear and can Kokkos be used to talk between processes, that was immediately clear based on my reading.

Closest to Astaroth is Parthenon[@grete_parthenonperformance_2023], which is a distributed framework for adaptive mesh refinement using Kokkos as the backend for intra-node computations.
In contrast, Astaroth provides a DSL and an optimizing code generator for implementing the computations akin to Halide, Polymage, and Patus.
Astaroth also incorporates other key functionalities for computational sciences, e.g., distributed reductions, IO, and supports different physics cases.

> TP: We have tried to remove words we can not defend or quantify like "ergonomic" and "compact". I would say now the word "modular" is one such word again. What do you think? I would be fine also with dropping this sentence since also we should not advertise different physics cases Astaroth comes since IMO they are not modular or expansive enough to advertise.

> OL: I think modular is arguably ok, if it can be shown. Unlike ergonomic or compact, modular has an objective definition: something that consists of modules which can be combined to form a working system. If example modules are listed and how they work together is explained, that would justify the term, IMO. But we may of course run up against the word count...

> OL: but maybe a better solution would be to talk about the components as "modules" (i.e. "different physics modules". "IO module"? "reduction module"?, idk which components are covered by the "modular") instead of describing the overall structure of the system as "modular"

> MV: Yes I agree with OL here. Some specificity can be a benefit. Astaroth has a number of different component both in and outside what is done in DSL. Now due to my work in AI I also tent to speak more about stuff in terms or "orchestration" and "workflow" too with respect to utilizing varius code components. Not sure if those word would be useful in this text. Here I am merely thinking out loud.   

> TP: Right, you guys make good points. Although I still have a worry about is saying the physics features are modular. If there are not developments I am not aware of the physics choices are driven by the macro flags. This makes the implementation clearly switchable, but not sure can we call it modular. Modules are strongly about interfaces and hiding implementation details to have interchangeable implementations that one can choose between. The switch based implementation neither hides details or has well defined interfaces that enable switchable implementations. (And not having really options to choose between many of the physics aspects like equation of state furthers my worry, but this maybe is asking for too much for something to be modular). I have now tentatively dropped the word


A distinctive feature of Astaroth is its specialization for cache-constrained use cases, especially in multiphysics simulations where the values of interdependent fields need to be held in working memory at the same time. Additionally, `Astaroth` does not only consider stencils in isolation, but also their combinations with other operations inside the same kernel.

# Software design

`Astaroth` consists of three main components: 1) `acc`, a compiler and runtime for a domain-specific language (DSL) for stencil computations, 2) an API for executing stencil applications on multi-GPU platforms, and 3) a standalone solver for certain simulation cases.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

> MR: I sum up on "runtime": Two terms are used in CS literature, with same meaning. One is of clear language and sufficiently self-explaining, the other is ambiguous jargon. It should be clear which to employ. If no consensus, majority will decide.

> JP: "a standalone solver" suggest "standalone solvers. Depends on our definition of a solver (meaning RK3 or MHD&TFM&etc here?).

## `acc` compiler and runtime

`Astaroth` has a DSL for stencil-based computation, designed to be used by domain scientists without having to consider technical implementation details.
The main operations, like stencils, are written in a declarative syntax, and the kernels that use them are written in an imperative syntax. [^paradigm_footnote]
The implementation of the operations is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. 
Of central importance of these is the unrolled and reordered computation of all required stencils at the start of the kernels, which enables instruction-level parallelism and efficient usage of caches [@pekkila_graphicsprocessors_2026].
In addition to stencils, the DSL supports two other operations: 1) reductions -- which are commonly needed for stencil-based solvers and require multiple steps to perform across multiple GPUs, and 2) distributed simplified ray-tracing, where rays cannot change directions and are restricted to move through neighbouring grid points -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].
Additionally, `Astaroth` comes with its own standard library for the DSL: It provides, in addition to other functionality, derivative operators needed for PDE solvers, which are implemented for generally spaced Cartesian, spherical or cylindrical grids, and Poisson solvers needed for, e.g. self-gravity.

> MR: I reiterate on declarative vs. imperative: There is no surplus in these terms for domain scientists - they are simply not interesting for them. Again a majority decision case.
> TP: Okay fine, but we can make concessions to the more technical readers at certain places? As Oskar has pointed out earlier we are talking about quite technical things here anyways (compilers and different ways of compilation).

> MR: "software-managed caches" this could be misread as, e.g., use of shared memory
> TP: Maybe but if the user really wants to be sure they can read the thesis

`acc` transpiles the DSL source into CUDA or HIP source code, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc` runtime, which further optimizes the kernels by autotuning the thread block sizes for kernel execution.
`acc` also supports conditional compilation based on any conditional statements in the DSL source --- only compiling code that is actually executed at run-time.
Further, `acc` supports run-time compilation, because run-time configuration parameters may change the evaluation of those conditional statements.
The information thus gained also allows `Astaroth` to optimize run-time behaviour more precisely, e.g. memory allocations or communication patterns.

> TP: The only worry is that is the text understandable enough for the targeted physicist readers so viewpoints from Miikka,Sienny,Maarit and Matthias would be valuable!
> MR: "conditional compilation" refers to the preprocessor conditionals, right? But if so, isn't this a very standard functionality, not needed to be stressed?
> TP: I understand that Oskar groups code elimination also to conditional compilation, which is correct based on my understanding of the words definition.

> MV: I am fine with the text here. I think however Sienny insights could be helpful here, as she is less familiar with the intimate details of how this all works. My view might be "contaminanted" by the discussion we already had. 


## Multi-GPU runtime API

In the DSL, users can define a list of compute steps specifying which kernels to run and which boundary conditions to impose.
`Astaroth`'s multi-GPU runtime constructs a directed acyclic graph (DAG) of the compute steps, where each step is decomposed into computation and communication tasks.
The decomposition into tasks is based on the overall domain decomposition, and the stencils' data access patterns, which also determines dependency relations between the tasks.
As an optimization, kernels may be fused together to reduce memory reads.

> TP: What do you think is it worth mentioning that the system drops unnecessary calls i.e. those without observable effect due to configuration variables? If yes, then I would propose the following: "As an optimization, unecessary kernel calls are dropped and to reduce memory reads kernels may be fused together."
> OL: that can be mentioned if there are words left in the budget. But a "call" is ambiguous". A call to what? A kernel? Needs to be specified.
> TP: Yes, calls to kernels. Modified the suggestion to reflect this

> JP: "fused together to reduce memory reads" a bit ambiguous. Is this done automatically? The integration kernels are fused to exploit the interdependence of the fields which does reduce memory reads. But the fusion of packing and reduction operations is done for better efficiency (small problem sizes fused to saturate the device with work). Should clarify what is meant here.

> TP: This refers to a bit niche implementation I did where the DSL compiler infers which kernels would be good candidates for fusion and automatically generates fused versions of them and at runtime decides can it use them. The motivation is that then the user can write Kernels that have smaller better defined scopes but can be then fused together by the runtime. I would be fine if the wording is a bit ambiguous since we cannot exactly describe what is done in a few words anyways.

`Astaroth`'s task scheduler executes these DAGs, asynchronously launching computation and communication tasks as prerequisite tasks are completed.
This improves performance in communication-bound cases, especially for higher process counts [@lappi2021task].
For fast data transfers and to support all possible hardware, both GPU-to-GPU remote direct memory access (RDMA) and CPU-to-CPU communication are supported.

This runtime system can be accessed through `Astaroth`s runtime API.
The API is C-ABI compatible, supporting foreign function interfaces to external applications written in any programming language.
The API is organized into two layers: the `Device` layer and the `Grid` layer.
The `Device` layer provides access to single-GPU functionality, such as moving data between CPU and GPU, launching kernels, and loading/storing snapshots from/to disk.
The `Grid` layer provides access to multi-GPU functionality, such as executing DAGs, distributed initialization, and distributed loading/storing of snapshots.
Other special functionality is also provided through the API, such as distributed Fourier transforms.

## Solver

`Astaroth` also includes a standalone finite-difference solver, which takes full advantage of the DSL and runtime API, and can be used to write new simulations.
It also works as a testbed for performance research.
This solver uses an astrophysical magnetohydrodynamical setup (`acc-runtime/samples/mhd_modular`) by default, but can be configured to run any DSL code.
The samples directory also includes other production-ready setups, e.g. `tfm-mpi` for the test-field method [@johannes_paper].

> JP: @johannes_paper will probably not get a doi before May-June so can reference the dsc instead where it's embedded
> TP: What about arxiv? You could but it there and we could refer to it from there?

The solver takes care of distributed initial conditions, domain decomposition, simulation diagnostics, and logging.
It is also built to react to a number of events, such as NaNs in the simulation data, simulation time limits, and a stop signal given through the file system.
The folder `analysis/` contains Python-based data analysis tools, which can be used to process and work with the data produced by the standalone solver. 

> JP: solver definition a bit unclear throughout the article. Do we mean the finite-diff + RK3 solver, or MHD/TFM/etc? Should pick one and use it throughout.
> TP: I think the full physics solver is meant throughout. Would not immediately come up with a way to improve it. But at the same time I am not sure is it that unclear (we of course rely on the reader to be familiar with how the word solver is usually used in this context, but that is IMO fine).

> OL: I think the test-field methods may be best expanded in research impact. I've mentioned them here for now. 

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. 
Additionally it has been used for research on performance optimization methods[@pekkila_graphicsprocessors_2026;@pekkila2025stencil;@pekkila2017methods], communication techniques [@pekkila2022scalable;@lappi2021task], compiler techniques[@pekkila_masters_2019;@puro2023programmatic] and other topics [@yokelson2024soma; @puro2025gpu].
We expect that the acceleration of `Pencil Code`, by integrating `Astaroth`'s DSL and runtime inside of it, will increase the number of `Astaroth` users.
The associated performance increase of 20-60x will enable more realistic astrophysical simulations in a wide range of use cases from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

> OL: Edit suggestion added for penultimate sentence, old sentence below. "People relying on" is fuzzy, would prefer simply "users". Also used active voice to make it clear who it is that expects this migration to happen. 
> TP: Yes agree that wording was fuzzy. The reason why I did not use the word users was that I was not 100% sure what constitutes a user of Astaroth when they do not directly interact with it, or are not necessarily are aware of its existence, but maybe my worry was overblown.
> OL: Ok, so then this sentence is indeed about the PCA interface. In that case I feel we really do need to mention it. Because otherwise, if e.g. a pencil code user reads this paper, the only reference to solvers is the standalone solver, and I think that is unhelpful. It can be mentioned as work in progress.
> TP: I get the point now: we have to make it clear PC is not accelerated via the standalone solver. Have to think about better wording but now made the tentative change: "... Pencil Code via Astaroth ... " ---> ".. Pencil Code, by integrating Astaroth's DSL and runtime inside of it, ... , ...". Maybe not the best but makes it more clear how Astaroth is used. I particularly think the inside of it is a bit clunky but for now this tentative version is better.

> OL: I agree with Johannes about more references, but don't really care which way they are listed. Either way is fine, although I think the gravitational waves paper was PC not Astaroth.

> TP: I have tried to give all references that now come to mind, including those in Johannes' suggestion. If you spot one is missing please simply add it. Yes, the gravitational waves paper is for PC and to showcase what will be done in the future with PC.

> OL: should there be more highlights on the domain science side? Currently the only highlight is the performance
> TP: What more should we say? We can say something in general terms but would we speak about new physics results like the asymptotics discovered in Fred's paper? But again there the only meaningful role Astaroth played was the performance, naturally.

> OL: The pencil code accelleration is mentioned without explanation of the PCA setup. I read it as referring to this development, it should be expanded to explain this to the reader (not a lot of text, just one or two sentences).

> OL: to me, the change that would drive users to Astaroth is the PCA transpiler method, as that allows PC users to keep their own methods. I don't think PC users will be migrating to the standalone solver. But as long as we make it clear that this is just an expectation that WE have, I guess it's fine. Made an edit suggestion about this above.

> TP: Do you read the text now as that we say we are expecting people to migrate to the standalone solver? Agreed that will not happen (at least from the PC community) so that is not we are trying to convey. Have now worded the text to be clear that Astaroth is inside PC and we are not accelerating it with the standalone solver.

# Acknowledgements

We acknowledge the contributions of all developers of Astaroth[^contributor_footnote] and also the early users of it who have been instrumental in its evolution. These include Jörn Warnecke, Frederick Gent, Ruben Krasnopolsky, Wei-Wen Li and Indrani Das.
We acknowledge the computational resources and services provided by CSC — IT Center for Science, the Aalto Science-IT project, ASIAA High-Performance Computing, and National Center for High-Performance Computing (NCHC), National Applied Research Laboratories (NARLabs) in Taiwan, the Oak Ridge Leadership Computing Facility at the Oak Ridge National Laboratory, and resources from LUMI-G through the Euro-HPC joint undertaking. Furthermore, we appreciate the important technical assistance provided by CSC, by people like Fredrik Robertsén and others.
The development of `Astaroth`  has received funding from the Academy of Finland, ReSoLVE Centre of Excellence, Grant/Award Number: 307411;
The European Research Council, the European Union's Horizon 2020 research and innovation program, project UniSDyn, Grant/Award Number: 818665; KAUTE Foundation, Grant/Award Numbers: 20240173 and 20250154.

> TP: Would Maarit know the best which funding sources to cite for the development of Astaroth??

> MV: Good question! What is the policy here? How far we should go? Like should I mention Wihuri and SKR because I got funding from then for my PhD?

> TP: Would leave it to Maarit to know best which ones to mention. The JOSS paper for PC has approx ~5 sources mentioned so we should definitely not need more than that so maybe the 3 most important?


> MV: Similarly question to Sienny: should we add anyone or anything from Taiwan to the Acknowledgements, e.g. NCHC? Or notable funding sources. I tentatively added something about the computational services into the text.

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^paradigm_footnote]: In declarative programming, computations are defined by describing what the results look like; in imperative programming, by describing the steps to perform.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém and Tzu-Chun Hsu. 
