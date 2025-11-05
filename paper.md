---
title: "ODINN.jl: Scientific machine learning glacier modelling"
tags:
  - Julia
  - glaciers
  - climate
  - SciML
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

`ODINN.jl` is a glacier model leveraging scientific machine learning (SciML) methods to perform forward and inverse simulations of large-scale glacier evolution. It can simulate both surface mass balance and ice flow dynamics through a modular architecture which enables the user to easily modify model components. For this, `ODINN.jl` is in fact an ecosystem composed of multiple packages, each one handling a specific task:

- `Sleipnir.jl`: Handles all the basic types, functions and datasets, common through the whole ecosystem, as well as data management tasks.
- `Muninn.jl`: Handles surface mass balance processes, via different types of models. 
- `Huginn.jl`: Handles ice flow dynamics, by solving the ice flow partial differential equations (PDEs) using numerical methods. It can accommodate multiple types of ice flow models. 
- `ODINN.jl`: Acts as the interface to the whole ecosystem, and provides the necessary tools to differentiate and optimize any model component. It can be seen as the SciML layer, enabling different types of inverse methods, using hybrid models combining differential equations with data-driven models. 

Splitting large Julia [@bezanson_julia_2017] packages into smaller, focused subpackages is a good practice that enhances maintainability, usability, and collaboration. Modular design simplifies debugging, testing, and updates by isolating functionalities, while users benefit from faster precompilation and reduced memory overhead by loading only the subpackages they need. This approach also lowers the barrier for new contributors, fosters clearer dependency management, and ensures scalability as projects grow, ultimately making the ecosystem more robust and adaptable. The ODINN ecosystem extends beyond this suite of Julia packages, by leveraging the data preprocessing tools of the Open Global Glacier Model (OGGM, @maussion_open_2019). We do so via an auxiliary Python library named `Gungnir`, which is responsible for downloading all the necessary data to force and initialize the model, such as glacier outlines from the Randolph Glacier Inventory (@rgi_consortium_randolph_2023, RGI), digital elevation models (DEMs), ice thickness observations from GlaThiDa [@glathida_consortium_glacier_2020], ice surface velocities from different studies [@millan_ice_2022], and many different sources of climate reanalyses and projections [@lange_wfde5_2019; @eyring_overview_2016]. This implies that `ODINN.jl`, like OGGM, is virtually capable of simulating any of the ~274,000 glaciers on Earth [@rgi_consortium_randolph_2023]. 

![Overview of the ODINN.jl ecosystem. \label{fig:ecosystem}](figures/odinn_ecosystem_v4.png)

`ODINN.jl` provides a high-level user-friendly interface, enabling the user to swap and replace most elements of a glacier simulation in a very modular fashion. The main elements of a simulation, such as the `Parameters`, a `Model` and a `Simulation` (i.e. a `Prediction` or an `Inversion`), are all objects that can be easily modified and combined. In a few lines of code, the user can automatically retrieve all necessary information for most glaciers on Earth, compose a `Model` based on a specific combination of surface mass balance and ice flow models, and incorporate data-driven models (e.g. a neural network) to parametrize specific physical processes of any of these components. Both forward and reverse simulations run in parallel using multiprocessing, leveraging Julia's speed and performance. Graphics Processing Unit (GPU) compatibility is still not ready, due to the difficulties of making everything compatible with automatic differentiation (AD). Nonetheless, it is planned for future versions. 

The most unique aspect of `ODINN.jl` is its differentiability and capabilities of performing all sorts of different hybrid modelling. Since the whole ecosystem is differentiable, we can optimize almost any model component, providing an extremely powerful framework to tackle many scientific problems [@bolibar_universal_2023]. `ODINN.jl` can optimize, separately or together, in a steady-state or transient way:
  
- The initial or intermediate state of glaciers (i.e. their ice thickness) or the equivalent ice surface velocities.
- Model parameters (e.g. the Glen coefficient `A` related to ice viscosity in a 2D Shallow Ice Approximation [@hutter_theoretical_1983]), in a gridded or scalar format. This can be done for multiple time steps where observations (e.g. ice surface velocities) are available.
- The parameters of a regressor (e.g. a neural network), used to parametrize a subpart or one or more coefficients of an ice flow or surface mass balance mechanistic model. This enables the exploration of empirical laws describing physical processes of glaciers, leveraging Universal Differential Equations (UDEs, @rackauckas_universal_2021). 

For this, it is necessary to differentiate (that is, computing gradients or derivatives) through complex code, including numerical solvers, which is a non-trivial task [@sapienza_differentiable_2024]. We use reverse differentation based on the adjoint method to achive this. We have two strategies for computing both the adjoint and the required vector-jacobian products (VJPs): (1) manual adjoints, which have been implemented using AD via `Enzyme.jl` [@enzyme], as well as fully manual implementations of the discrete and continuous adjoints; and (2) automatic adjoints using `SciMLSensitivity.jl` [@rackauckas_diffeqfluxjl_2019], providing both continuous and discrete versions and available with different AD back-ends. These two approaches are complementary, with the manual adjoints being ideal for high-performance tasks, and serving as a ground truth for benchmarking and testing automatic adjoint methods from `SciMLSensitivity.jl`.

Beyond all these inverse modelling capabilities, `ODINN.jl` can also act as a more conventional forward glacier model, simulating glaciers in parallel, and easily customizing almost every possible detail of the simulation. Its high modularity, combined with the easy access to a vast array of datasets coming from OGGM, makes it very easy to run simulations, even with a simple laptop. `Huginn.jl` is responsible for the ice flow dynamics models, with an architecture capable of integrating and easily swapping various models. Models based on partial differential equations (PDEs) are solved using `DifferentialEquations.jl` [@rackauckas_differentialequationsjl_2017], which provides access to a huge amount of numerical solvers. For now, we have implemented a 2D Shallow Ice Approximation (SIA, @hutter_theoretical_1983), but in the future we plan to incorporate other models, such as the Shallow Shelf Approximation (SSA, @weis_theory_1999). Validation of numerical forward simulations are evaluated in the test suite based on exact analytical solutions of the SIA equation for some simpler cases [@Bueler_2005]. In terms of surface mass balance, `Muninn.jl` incorporates for now simple temperature-index models. Nonetheless, the main addition of the upcoming version will be the machine learning-based models from the `MassBalanceMachine` [@sjursen_machine_2025], which will become the de-facto solution. Frontal ablation (i.e. calving) and debris cover are not available for now, but we plan to add it to future versions of the model. 


# Statement of need

`ODINN.jl` addresses the need for a glacier model that combines the physical interpretability of mechanistic approaches with the flexibility and data-assimilation capabilities of data-driven methods [@bolibar_universal_2023]. By integrating both paradigms, it enables targeted inverse methods to learn parametrizations of glacier processes, capturing unknown physics while preserving the physically grounded structure of glacier dynamics through differential equations.

While purely mechanistic and purely data-driven glacier models already exist (e.g. @gagliardini_capabilities_2013, @maussion_open_2019, @rounce_global_2023, @bolibar_nonlinear_2022), they often lack the flexibility needed to fully exploit the growing wealth of glacier observations, such as ice surface velocities, ice thickness, surface topography, surface mass balance or climate reanalyses. Existing empirical laws do not always link directly to these observables, making their calibration challenging. Approaches based on differentiable programming and functional inversions offer a path forward, allowing the derivation of new empirical relationships from carefully chosen proxies and providing a framework to test hypotheses about poorly understood physical processes such as basal sliding, creep, or calving.

Improving the representation of these complex processes is crucial for accurate projections of glacier evolution and their impacts on freshwater availability and sea-level rise [@ipcc_climate_2021]. To this end, ODINN.jl provides a unified modelling ecosystem that supports both advanced inverse methods for model calibration and efficient, modular forward simulations for large-scale glacier studies.

Developing such a framework places demanding requirements on scientific software. Inefficient or monolithic implementations can hinder progress, emphasizing the importance of open-source, community-driven tools that follow modern software engineering practices. The Julia programming language provides two key advantages in this context: it resolves the two-language problem by offering Python-like high-level expressiveness with C-level performance [@bezanson_julia_2017], and it enables source-code differentiability, essential for gradient-based optimization in inverse modelling.

With ODINN.jl, our goal is to provide a robust and future-proof modelling environment that bridges the gap between physical understanding and data-driven discovery. Its modular architecture, thorough testing, and continuous integration (CI) ensure reproducibility and reliability, while its open design invites collaborations and both methodological and applied advancements across the glaciological and Earth system modelling communities.

# Acknowledgements

We acknowledge the help of Chris Rackauckas for the debugging and discussion of issues related to the SciML Julia ecosystem, Redouane Lguensat for scientific discussions on the first prototype of the model, and Julien le Sommer for scientific discussions around differentiable programming. JB acknowledges financial support from the Nederlandse Organisatie voor Wetenschappelijk Onderzoek, Stichting voor de Technische Wetenschappen (Vidi grant 016.Vidi.171.063) and a TU Delft Climate Action grant. FS acknowledges funding from the National Science Foundation (EarthCube programme under awards 1928406 and 1928374). AG acknowledges funding from the MIAI cluster and Agence Nationale de la Recherche (ANR) in the context of France 2030 (grant ANR-23-IACL-0006).

# References

