---
title: 'Astaroth: A scientific computing framework for accelerating stencil computations.'
tags:
  - GPU
  - DSL
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
While stencils are the core of `Astaroth`, it also accelerates other operations like reductions (e.g. sums), simple ray-tracing, and integrates with libraries performing GPU-accelerated Fourier transforms, all of which are important for simulations on structured grids.
`Astaroth` is optimized for multiphysics use cases and has primarily been used for turbulent astrophysical plasma simulations.


> MV: In this section we do not address at all that one of the big selling points for Astaroth is that it is very good for cache-constrained multiphysics. Right now there is no work about it in the summary, while it was a huge deal with respect how Astaroth works as good as it does. IMHO, even more important innovation from Johannes that the DSL. It deserves at least a sentence in the summary. 
> TP: Good point! We speak about this later and I would like to keep the more technical wording to that section so I know added simply that `Astaroth` is optimized for multiphysics use cases.

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

> MV: Another TODO item: we need to qualify the 20-60x speedup better in the text. Such metrics are very much contextual and could be an issue for the referee if we are not more precise. 
> TP: Would think that the immediate citation makes it clear that the context can be found in the paper for the reader who is interested. I would not think that JOSS expects to expand on speedup results. If the referees complain I would then expand on the context.
> MV: If it is like so, then OK. 


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
In contrast, Astaroth provides a DSL and an optimizing code generator for implementing the computations akin to Halide, Polymage, and Patus.
Astaroth also incorporates other key functionalities for computational sciences, e.g., distributed reductions, IO, and supports different physics cases.

A distinctive feature of Astaroth is its specialization for cache-constrained use cases, especially in multiphysics simulations where the values of interdependent fields need to be held in working memory at the same time. Additionally, `Astaroth` does not only consider stencils in isolation, but also their combinations with other operations inside the same kernel.

# Software design

`Astaroth` consists of three main components: 1) `acc`, a compiler and runtime system for a domain-specific language (DSL) for stencil computations, 2) an API for executing stencil applications on multi-GPU platforms, and 3) a standalone solver for certain simulation cases.
Below, we present a quick overview of these components. More extensive documentation is available at [@astaroth_doc]. 

## `acc` compiler and runtime system

`Astaroth` has a DSL for stencil-based computations, designed to be used by domain scientists without having to consider technical implementation details.
The main operations, like stencils, are written in a declarative syntax, and the kernels that use them are written in an imperative syntax. [^paradigm_footnote]
The implementation of the operations is left to `Astaroth`'s DSL compiler `acc`, which applies a number of specialized optimizations. 
Of central importance of these is the unrolled and reordered computation of all required stencils at the start of the kernels, which enables instruction-level parallelism and efficient usage of caches [@pekkila_graphicsprocessors_2026].
In addition to stencils, the DSL supports two other operations: 1) multi-GPU reductions -- which are commonly needed for stencil-based solvers, and 2) distributed simplified ray-tracing, where rays cannot change directions and are restricted to move through neighbouring grid points -- which is necessary for simulations incorporating radiative transfer [@heinemann2006radiative].

Additionally, `Astaroth` comes with its own standard library for the DSL: It provides, in addition to other functionality, derivative operators needed for PDE solvers, which are implemented for generally spaced Cartesian, spherical or cylindrical grids, and Poisson solvers needed for, e.g. self-gravity.

> MR: I reiterate on declarative vs. imperative: There is no surplus in these terms for domain scientists - they are simply not interesting for them. Again a majority decision case.
> TP: Okay fine, but we can make concessions to the more technical readers at certain places? As Oskar has pointed out earlier we are talking about quite technical things here anyways (compilers and different ways of compilation).

> MR: I talked to Fred (native speaker), he says "reductions, which ... require several steps to be performed ..." or
>                                                                               several steps    be performed ..."
>  "several" instead of "multiple" to avoid doubling and to clarify difference between "some" and "many"
> OL: reduced "multiple steps to be performed over multiple GPUs" to simply "multi-GPU". It conveys the fact that its complicated.

`acc` transpiles the DSL source into CUDA or HIP source code, which is further compiled into machine code using a native CUDA or HIP compiler.
The program thus produced is executed in the `acc` runtime system, which further optimizes the kernels by autotuning the thread block sizes for kernel execution.
`acc` also supports run-time compilation, because run-time configuration parameters may change the evaluation of conditional statements, thereby changing the branches taken at run-time.
With run-time compilation, for a given configuration, `acc` compiles only those parts of the DSL source that will be executed.
The information of what code gets executed also allows `Astaroth` to optimize run-time behaviour more precisely, e.g. memory allocations or communication patterns.

> OL: rewrote the paragraph based on the discussion on monday (May 4th). Removed reference to conditional compilation. Hope this is more clear.
> TP: The information thus gained ---> the information of what code gets executed, since the paragraph has underwent some changes and this is now clearer.

## Multi-GPU runtime system and API

In the DSL, using the `ComputeSteps` language construct, users can define a list of compute steps specifying a sequence of kernels and boundary conditions.
Kernels defined in `ComputeSteps` may be fused to reduce memory reads.
Based on the overall domain decomposition and the stencils' data access patterns, `acc` infers the dependency relationships between the steps, and constructs a directed acyclic graph (DAG) of dependent tasks.
Each step is split into tasks along these regions: the big region at the core of a subdomain --- which is not dependent on communicated data from neighbors, and the smaller regions at the boundaries --- which are.
Communication tasks are inserted where needed.

> TP: Boundary updates ---> boundary conditions, so the actual function of them is more apparent to the reader.

> TP: What do you think is it worth mentioning that the system drops unnecessary calls i.e. those without observable effect due to configuration variables? If yes, then I would propose the following: "As an optimization, unecessary kernel calls are dropped and to reduce memory reads kernels may be fused together."
> OL: that can be mentioned if there are words left in the budget. But a "call" is ambiguous". A call to what? A kernel? Needs to be specified.
> TP: Yes, calls to kernels. Modified the suggestion to reflect this

> JP: "fused together to reduce memory reads" a bit ambiguous. Is this done automatically? The integration kernels are fused to exploit the interdependence of the fields which does reduce memory reads. But the fusion of packing and reduction operations is done for better efficiency (small problem sizes fused to saturate the device with work). Should clarify what is meant here.

> TP: This refers to a bit niche implementation I did where the DSL compiler infers which kernels would be good candidates for fusion and automatically generates fused versions of them and at runtime decides can it use them. The motivation is that then the user can write Kernels that have smaller better defined scopes but can be then fused together by the runtime. I would be fine if the wording is a bit ambiguous since we cannot exactly describe what is done in a few words anyways.

> OL: Matthias wrote a suggestion for the above paragraph. I took some of his suggestion and added some of my own improvements. There's still the matter of "dropping unnecessary kernel calls". Now that I think about it, this will probably raise more questions. The reader will wonder why there would be unnecessary kernels in a `ComputeSteps`.

> TP: Fair enough. The motivation is similar to run-time compilation: some kernels are only meaningful depending on some input variables. I am fine with not mentioning the feature.

> JP: grammar confusing "core of a subdomain --- ... --- which are". 'Which are' refers to 'smaller regions' so not independent clauses. I would also avoid '---' throughout and stick solely to commas.

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
> MJKL: I second Touko in this. We should be able to make a re-submission in the beginning of the next week, after which it is also a high time to make an arxiv submission.

The solver takes care of distributed initial conditions, domain decomposition, simulation diagnostics, and logging.
It is also built to react to a number of events, such as NaNs in the simulation data, simulation time limits, and a stop signal given through the file system.
The directory `analysis/` contains Python-based data analysis tools, which can be used to process and work with the data produced by the standalone solver. 

> JP: solver definition a bit unclear throughout the article. Do we mean the finite-diff + RK3 solver, or MHD/TFM/etc? Should pick one and use it throughout.
> TP: I think the full physics solver is meant throughout. Would not immediately come up with a way to improve it. But at the same time I am not sure is it that unclear (we of course rely on the reader to be familiar with how the word solver is usually used in this context, but that is IMO fine).
> MV: I would qualify something as a "solver" if it applies Astaroth to its intended purpose to solve the inteded problem. With this I mean not only FDM + RK3, but required setting, orchestration and boundary condition. E.g full MHD solver. Just limiting to FDM + RK3 is, in my opinion, overtly reductive, because that would not be how the userbase would think. 
> TP: Agreed, and again not sure how we could make the wording less ambiguous to mean the full physics solver.

> OL: I think the test-field methods may be best expanded in research impact. I've mentioned them here for now.

> HS: It might be helpful to distinguish HD/MHD/Physics solvers. It may be good to add the existing physics solvers and capabilities.

> HS: One possibility is to also add the status on high-order solvers in the field?

> OL: that could be valuable. We could at least mention them. Do you know what the main ones are?

> MV: Oh dear. To answer this question properly will require more time that I have.  

# Research impact statement

`Astaroth` has already been used in many papers as the core PDE-solver, mainly for astrophysical plasma simulations [@vaisala2021interaction; @vaisala2023exploring; @gent2026asymptotic], but also in seismology [@ladino2025acoustic]. 
Additionally it has been used for research on performance optimization methods[@pekkila_graphicsprocessors_2026;@pekkila2025stencil;@pekkila2017methods], communication techniques [@pekkila2022scalable;@lappi2021task], compiler techniques[@pekkila_masters_2019;@puro2023programmatic] and other topics [@yokelson2024soma; @puro2025gpu].
We expect that the recent acceleration of `Pencil Code`, which was done by embedding `Astaroth`'s DSL and runtime system inside of it, will increase the number of `Astaroth` users.
The associated speedup of 20-60x will enable more realistic astrophysical simulations in a wide range of use cases from modelling small-scale dynamos [@warnecke2025small] to the propagation and processes producing primordial gravitational waves [@roper2020numerical].

> OL: should there be more highlights on the domain science side? Currently the only highlight is the performance
> TP: What more should we say? We can say something in general terms but would we speak about new physics results like the asymptotics discovered in Fred's paper? But again there the only meaningful role Astaroth played was the performance, naturally.

> MV: This comment here is just a TODO note to myself that I need to find a way to address this better from my side when I can. Also does the gravitational wave paper use Astaroth? If not, we need to be more clear about it.  
> TP: Doesn't the word will in the sentence make it clear that we are speaking about future work that has not been done yet and thus has not used Astaroth yet? 
> MV: Sure now that I think about it again. 

# Acknowledgements

We acknowledge the contributions of all developers of Astaroth[^contributor_footnote] and also the early users of it who have been instrumental in its evolution. These include Jörn Warnecke, Frederick Gent, Ruben Krasnopolsky, Wei-Wen Li, Mordecai Mac Low, Chun-Fan Liu, Man Hei Li and Indrani Das.
We acknowledge the computational resources and services provided by CSC — IT Center for Science, the Aalto Science-IT project, ASIAA High-Performance Computing, and National Center for High-Performance Computing (NCHC), National Applied Research Laboratories (NARLabs) in Taiwan, the Oak Ridge Leadership Computing Facility at the Oak Ridge National Laboratory, and resources from LUMI-G through the Euro-HPC joint undertaking. Furthermore, we appreciate the important technical assistance provided by CSC, by people like Fredrik Robertsén and others.
The development of `Astaroth`  has received funding from the Academy of Finland, ReSoLVE Centre of Excellence, Grant/Award Number: 307411;
The European Research Council, the European Union's Horizon 2020 research and innovation program, project UniSDyn, Grant/Award Number: 818665; KAUTE Foundation, Grant/Award Numbers: 20240173 and 20250154; Research Council of Finland, project MomEnt, Grant/Award Number: 373416.
The authors acknowledge support for the CompAS Project from the Institute of Astronomy and Astrophysics, Academia Sinica (ASIAA), the Academia Sinica grant AS-IAIA-114-M01, and the National Science and Technology Council (NSTC) in Taiwan through grants 112-2112-M-001-030, 113-2112-M-001-008, and 114-2112-M-001-001-; the International Collaboration and Cooperation grant for COSMAGG that supports the exchanges between Taiwan and Finland: 113-2927-I-001-513-, 114-2927-I-001-506-, and Research Council of Finland project 359462.

> TP: Would Maarit know the best which funding sources to cite for the development of Astaroth??

> MV: Good question! What is the policy here? How far we should go? Like should I mention Wihuri and SKR because I got funding from then for my PhD?

> TP: Would leave it to Maarit to know best which ones to mention. The JOSS paper for PC has approx ~5 sources mentioned so we should definitely not need more than that so maybe the 3 most important?


> MV: Similarly question to Sienny: should we add anyone or anything from Taiwan to the Acknowledgements, e.g. NCHC? Or notable funding sources. I tentatively added something about the computational services into the text.

> HS: The authors acknowledge support for the CompAS Project from the Institute of Astronomy and Astrophysics,
Academia Sinica (ASIAA), the Academia Sinica grant AS-IAIA-114-M01, and the National Science and Technology Council (NSTC) in Taiwan through grants 112-2112-M-001-030, 113-2112-M-001-008, and 114-2112-M-001-001-; the International Collaboration and Cooperation grant for COSMAGG that supports the exchanges between Taiwan
and Finland: 113-2927-I-001-513-, 114-2927-I-001-506-, and Research Council of Finland project 359462.

> HS: adding Mordecai Mac Low, Chun-Fan Liu, and Man Hei Li.

> MV: I added Sienny's references to the Acknowledgements properly now.

# References

[^stencil_footnote]: Stencil computations, or so called iterative stencil loops [@li2004automatic], are computations on structured grids where a given point is updated using a fixed neighborhood pattern. Good examples are convolutions in image processing and convolutional neural networks, and different schemes for spatial derivatives like the finite-difference method.
[^paradigm_footnote]: In declarative programming, computations are defined by describing what the results look like; in imperative programming, by describing the steps to perform.
[^contributor_footnote]: Contributors not otherwise credited in the text are: Petr Bém and Tzu-Chun Hsu. 
