---
title: "ODINN.jl: Scientific machine learning glacier modelling"
tags:
  - Julia
  - glaciers
  - climate
  - SciML
  - differentiable programming
  - hydrology
authors:
  - name: Jordi Bolibar
    orcid: 0000-0002-0791-0731
    affiliation: "1, 2" # (Multiple affiliations must be quoted)
    corresponding: true
  - name: Facundo Sapienza
    orcid: 0000-0003-4252-7161
    affiliation: 2
    affiliation: "3, 4"
    corresponding: true
  - name: Alban Gossard
    affiliation: 1
  - name: Mathieu le Séac'h
    affiliation: 1
  - name: Lucille Gimenes 
    affiliation: 1
  - name: Vivek Gajadhar 
    affiliation: 2
  - name: Fabien Maussion
    affiliation: "5, 6"
  - name: Ching-Yao Lai
    affiliation: 3
  - name: Bert Wouters
    affiliation: 2
  - name: Fernando Pérez
    orcid: 0000-0002-1725-9815
    affiliation: 4
affiliations:
 - name: Univ. Grenoble Alpes, CNRS, IRD, G-INP, Institut des Géosciences de l’Environnement, Grenoble, Franc
   index: 1
 - name: Faculty of Civil Engineering and Geosciences, Delft University of Technology, Delft, The Netherlands
   index: 2
 - name: Department of Geophysics, Stanford University, Stanford, United States
   index: 3
 - name: Department of Statistics, University of California, Berkeley, United States
   index: 4
 - name: Bristol Glaciology Centre, School of Geographical Sciences, University of Bristol, Bristol, UK
   index: 5
 - name: Department of Atmospheric and Cryospheric Sciences, University of Innsbruck, Innsbruck, Austria
   index: 6
date: 1 December 2025
bibliography: paper.bib

---

# Summary

`ODINN.jl` is a glacier model leveraging scientific machine learning (SciML) methods to perform forward and inverse simulations of large-scale glacier evolution. It can simulate both surface mass balance and ice flow dynamics through a modular architecture which enables the user to easily modify model components. 

The most unique aspect of `ODINN.jl` is its differentiability and capabilities of performing all sorts of different hybrid modelling. Since the whole ecosystem is differentiable (where differentiable means the ability to compute model derivatives with respect to parameters [@Shen_2023]), we can optimize almost any model component, providing an extremely powerful framework to tackle many scientific problems [@bolibar_universal_2023]. `ODINN.jl` can optimize, separately or together, in a steady-state (time-independent simulation) or transient (time-dependent simulation) way the following model parameters:

- The initial or intermediate state of glaciers (i.e. their ice thickness) or the equivalent ice surface velocities.
- Model parameters (e.g. the Glen coefficient `A` related to ice viscosity in a 2D Shallow Ice Approximation [@hutter_theoretical_1983]), in a gridded or scalar format. This can be done for multiple time steps where observations (e.g. ice surface velocities) are available.
- The parameters of a statistical regressor (e.g. a neural network), used to parametrize a subpart or one or more coefficients of an ice flow or surface mass balance mechanistic model. This enables the exploration of empirical laws describing physical processes of glaciers, leveraging Universal Differential Equations (UDEs, @rackauckas_universal_2021).

For this, it is necessary to differentiate (that is, computing gradients or derivatives) through complex code, including numerical solvers, which is a non-trivial task [@sapienza_differentiable_2024].
We use reverse differentiation based on the adjoint method to achieve this.
We have two strategies for computing both the adjoint and the required vector-jacobian products (VJPs): (1) manual adjoints, which have been implemented using AD via `Enzyme.jl` [@enzyme], as well as fully manual implementations of the spatially discrete and spatially continuous VJPs; and (2) automatic adjoints using `SciMLSensitivity.jl` [@rackauckas_diffeqfluxjl_2019], available with different AD back-ends for the VJPs computation.
These two approaches are complementary, with the manual adjoints being ideal for high-performance tasks by providing more control on the implementation, and serving as a ground truth for benchmarking and testing automatic adjoint methods from `SciMLSensitivity.jl`.

Beyond all these inverse modelling capabilities, `ODINN.jl` can also act as a more conventional forward glacier model, simulating glaciers in parallel, and easily customizing different model parametrizations and choices within the simulation. Its high modularity, combined with the easy access to a vast array of datasets coming from OGGM, makes it very easy to run simulations, even with a simple laptop. Multiple ice flow dynamics models can be easily swapped, thanks to a modular architecture (see Software design). Models based on partial differential equations (PDEs) are solved using `DifferentialEquations.jl` [@rackauckas_differentialequationsjl_2017], which provides access to a huge amount of numerical solvers. For now, we have implemented a 2D Shallow Ice Approximation (SIA, @hutter_theoretical_1983), but in the future we plan to incorporate other models, such as the Shallow Shelf Approximation (SSA, @weis_theory_1999). Validation of numerical forward simulations are evaluated in the test suite based on exact analytical solutions of the SIA equation [@Bueler_2005]. Multiple surface mass balance models are available, based on simple temperature-index models. Nonetheless, the main addition of the upcoming version will be the machine learning-based models from the `MassBalanceMachine` [@sjursen_machine_2025], which will provide further mass balance models.


# Statement of need

`ODINN.jl` addresses the need for a glacier model that combines the physical interpretability of mechanistic approaches with the flexibility and data-assimilation capabilities of data-driven methods [@bolibar_universal_2023]. By integrating both paradigms, it enables targeted inverse methods to learn parametrizations of glacier processes, capturing unknown physics while preserving the physically grounded structure of glacier dynamics through differential equations.

While purely mechanistic and purely data-driven glacier models already exist (e.g. @gagliardini_capabilities_2013, @maussion_open_2019, @rounce_global_2023, @bolibar_nonlinear_2022), they often lack the flexibility needed to fully exploit the growing wealth of glacier observations, such as ice surface velocities, ice thickness, surface topography, surface mass balance or climate reanalyses. Existing empirical laws do not always link directly to these observables, making their calibration challenging. Approaches based on differentiable programming and functional inversions offer a path forward, allowing the derivation of new empirical relationships from carefully chosen proxies and providing a framework to test hypotheses about poorly understood physical processes such as basal sliding, creep, or calving.

Improving the representation of these complex processes is crucial for accurate projections of glacier evolution and their impacts on freshwater availability and sea-level rise [@ipcc_climate_2021].
To this end, ODINN.jl provides a unified modelling ecosystem that supports both advanced inverse methods for model calibration and efficient, modular forward simulations for large-scale glacier studies.

Developing such a framework places demanding requirements on scientific software.
Inefficient codes and irreproducible implementations can severely restrict progress, emphasizing the importance of open-source, community-driven tools that follow modern research software engineering (RSE) practices [@Combemale_2023].
For this end, software should support modular and adaptable design patterns that enable prototyping and augmentation of existing pipelines [@Nyenah_2024].
The Julia programming language provides two key advantages in this context: it solves the two-language problem by offering Python-like high-level expressiveness with C-level performance [@bezanson_julia_2017], and it enables source-code differentiability, essential for modular gradient-based optimization in inverse modelling and setting the foundation for a strong ecosystem where hybrid modelling, and particularly UDEs, can thrive.
With ODINN.jl, our goal is to provide a robust and future-proof modelling framework that bridges the gap between physical understanding and data-driven discovery.
Its modular architecture, thorough testing, and continuous integration (CI) ensure reproducibility and reliability, while its open design invites collaborations and both methodological and applied advancements across the glaciological and Earth system modelling communities.

![Overview of the ODINN.jl ecosystem. \label{fig:ecosystem}](figures/odinn_ecosystem_v4.png)

# Software design

`ODINN.jl` is in fact an ecosystem composed of multiple packages, each one handling a specific task:

- `Sleipnir.jl`: Handles all the basic types, functions and datasets, common through the whole ecosystem, as well as data management tasks.
- `Muninn.jl`: Handles surface mass balance processes, via different types of models.
- `Huginn.jl`: Handles ice flow dynamics, by solving the ice flow partial differential equations (PDEs) using numerical methods. It can accommodate multiple types of ice flow models.
- `ODINN.jl`: Acts as the interface to the whole ecosystem, and provides the necessary tools to differentiate and optimize any model component. It can be seen as the SciML layer, enabling different types of inverse methods, using hybrid models combining differential equations with data-driven models.

Splitting large Julia [@bezanson_julia_2017] packages into smaller, focused subpackages is a good practice that enhances maintainability, usability, and collaboration.
Modular design simplifies debugging, testing, and updates by isolating functionalities, while users benefit from faster precompilation and reduced memory overhead by loading only the subpackages they need.
This approach also lowers the barrier for new contributors, fosters clearer dependency management, and ensures scalability as projects grow, ultimately creating a robust and adaptable software ecosystem.
The ODINN ecosystem extends beyond this suite of Julia packages, by leveraging the data preprocessing tools of the Open Global Glacier Model (OGGM, @maussion_open_2019).
We do so via an auxiliary Python library named `Gungnir`, which is responsible for generating all the necessary data to force and initialize the model, such as glacier outlines from the Randolph Glacier Inventory (@rgi_consortium_randolph_2023, RGI), digital elevation models (DEMs), ice thickness observations from GlaThiDa [@glathida_consortium_glacier_2020], ice surface velocities from different studies [@millan_ice_2022], and different sources of climate reanalyses and projections [@lange_wfde5_2019; @eyring_overview_2016].
This implies that `ODINN.jl`, like OGGM, is virtually capable of simulating any of the ~274,000 glaciers on Earth [@rgi_consortium_randolph_2023]. 

`ODINN.jl` provides a high-level user-friendly interface, enabling the user to swap and replace most elements of a glacier simulation in a very modular fashion. The main elements of a simulation, such as the `Parameters`, a `Model` and a `Simulation` (i.e. a `Prediction` or an `Inversion`), are all objects that can be easily modified and combined. In a few lines of code, the user can automatically retrieve all necessary information for most glaciers on Earth, compose a `Model` based on a specific combination of surface mass balance and ice flow models, and incorporate data-driven models (e.g. a neural network) to parametrize specific physical processes of any of these components. Both forward and inverse simulations run in parallel using multiprocessing, leveraging Julia's speed and performance. Graphics Processing Unit (GPU) compatibility is still not ready, due to the difficulties of making GPU architectures compatible with automatic differentiation (AD). Nonetheless, it is planned for future versions.

# Research impact statement

`ODINN.jl` has evolved through the last 5 years with code contributions during 3 postdoc positions, 1 PhD and 3 master internships. It has so far been used to explore the use of UDEs to invert hidden empirical laws in a synthetic glacier setup, where a prescribed rheological law was successfully recovered using a neural network [@bolibar_universal_2023]. This proof-of-concept then served as a backbone to create the current complex architecture of `ODINN.jl`, finalized with the recent 1.0 release. The main changes and scientific goals of this large software development investment, are the capacity to now apply these methods to large-scale remote sensing data for multiple glaciers, which will enable the exploration of new glacier basal sliding laws directly from heterogeneous observations, which remains a long-standing problem in glaciology [@minchew_toward_2020]. The development of the differentiable programming methods in `ODINN.jl`, also served as a catalyst to write an exhaustive review paper, together with other key players in this community, on differentiable programming for differential equations [@sapienza_differentiable_2024]. 

Additionally, `ODINN.jl` will be soon used as part of a newly funded 4-year project, to simulate past and future glacier changes in several catchments in the Andes and the Alps. These model outputs will then be combined with a hydrological model, to investigate the impacts of glacier retreat on the hydrological regimes and drought mitigation under different climate change scenarios. 

With these two research venues, `ODINN.jl` will be developed and used to pursue both fundamental research on glacier sliding laws, as well as applied research to assess the impact of glacier retreat on freshwater availability and drought mitigation. 

# AI usage disclosure

Generative AI, via GitHub copilot, has been used to partially generate some of the docstrings for the documentation, and to assist in the coding of some tests and helper functions. 


# Acknowledgements

We acknowledge the help of Chris Rackauckas for the debugging and discussion of issues related to the SciML Julia ecosystem, Redouane Lguensat for scientific discussions on the first prototype of the model, and Julien le Sommer for scientific discussions around differentiable programming. 
We thank all the developers of the SciML Julia ecosystem who work in each one of the core libraries used within `ODINN.jl`. 
JB acknowledges financial support from the Nederlandse Organisatie voor Wetenschappelijk Onderzoek, Stichting voor de Technische Wetenschappen (Vidi grant 016.Vidi.171.063) and a TU Delft Climate Action grant. 
FS and CYL were supported by NSF via grant number OPP-2441132 and the Alfred P. Sloan Foundation under grant number FG-2024-21649.
FS and FP acknowledges funding from the National Science Foundation (EarthCube programme under awards 1928406 and 1928374).
AG acknowledges funding from the MIAI cluster and Agence Nationale de la Recherche (ANR) in the context of France 2030 (grant ANR-23-IACL-0006).

# References

