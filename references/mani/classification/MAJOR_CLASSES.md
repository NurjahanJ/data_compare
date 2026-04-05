# Major Classes (Ferrofluids)

These major classes are derived from the scheme in `FEROFLUIDS_CLASS_DEFINITION.md`. They are intended as high-signal labels for a classification model.

## MATERIAL
**Definition:** Work focused on the ferrofluid material itself—its composition, formulation, intrinsic properties, and how it is prepared, handled, manipulated, or characterized.

**Include when the paper is primarily about:**
- Chemistry, constituents, synthesis routes, surfactants/coatings, nanoparticle composition
- Formulation/processing (recipes, concentrations, additives), stability of the formulation as a material property
- Material properties (magnetic, rheological, thermal, optical, colloidal, etc.)
- Evaluation / characterization methods and measurement results (e.g., magnetization curves, viscosity measurements, microscopy, spectroscopy)
- Material-level manipulation/handling topics (droplets as a material phenomenon, handling/storage/dispersion issues)

**Exclude when it is primarily about:**
- Numerical simulation / theoretical modelling as the main contribution → **COMPUTATION**
- Building/validating a device/system use-case (e.g., bearing, hyperthermia system) where the ferrofluid is mainly a component → **APPLICATION**
- Experimental demonstrations that are not primarily characterization of the material (e.g., lab experiments on systems/flows) → **EXPERIMENTATION**

## COMPUTATION
**Definition:** Work where the main contribution is computation, numerical simulation, or mathematical modelling of ferrofluid behavior (fields, flows, heat, stability, droplets), including tool- or method-centered studies.

**Include when the paper is primarily about:**
- FEA / FEM, CFD, numerical solvers, MATLAB-based simulation workflows
- Mathematical modelling: ODEs, boundary value problems, two-phase models, magneto-optical models
- Models of forces/effects (e.g., lifting force, dipole interactions, strain energy, susceptibility-based modelling)
- Simulated/analytical studies of flow, heat transfer, stability
- Simulations/models of magnetic droplets and droplet spin

**Exclude when it is primarily about:**
- Measuring/characterizing real ferrofluid samples (even if data is fit to a model) → **MATERIAL**
- Physical lab experiments (apparatus, experimental setup, measured outcomes) as the main evidence → **EXPERIMENTATION**
- A device/system application as the primary focus (even if it uses simulation as support) → **APPLICATION**

## EXPERIMENTATION
**Definition:** Work centered on physical experiments involving ferrofluids that are **not** primarily material characterization. This covers experimental demonstrations, measurements, and validations of phenomena, flows, devices, or setups.

**Include when the paper is primarily about:**
- Experimental setups, protocols, test rigs, and measured outcomes beyond basic material characterization
- Bench-top demonstrations of ferrofluid phenomena (e.g., behavior in fields/flows) where the novelty is experimental evidence
- Experimental validation of theories/simulations (when the experimental work is central)

**Exclude when it is primarily about:**
- Characterizing the ferrofluid material itself (evaluation/characterization as the core contribution) → **MATERIAL**
- Purely numerical/theoretical work without central experimental results → **COMPUTATION**
- An end-use device/system and its performance in a real application domain (medical/robotics/etc.) → **APPLICATION**

## APPLICATION
**Definition:** Work focused on using ferrofluids in an end-use application, device, or system (often with performance metrics, design constraints, or domain-specific requirements).

**Include when the paper is primarily about:**
- Systems based on magnetic induction principles
- Medical/biomedical applications: robotic surgery, hyperthermia, cancer therapy, drug delivery, pharmaceutical uses
- Robotics (general)
- Geology / oil-field recovery
- Application-oriented flow or heat transfer
- Engineering components: bearings, seals, lubricants
- Levitation or droplet spin as an application/system
- Digital microfluidics, damping, environmental/engineering uses
- Instruments whose purpose is to evaluate/operate with ferrofluids in an application context

**Exclude when it is primarily about:**
- Developing the ferrofluid composition/properties for their own sake → **MATERIAL**
- General modelling/simulation without an application-system focus → **COMPUTATION**
- General lab experimentation without an application-domain framing → **EXPERIMENTATION**

## REVIEW / BOOK
**Definition:** Secondary literature that summarizes, surveys, or compiles knowledge rather than presenting a new primary research contribution.

**Include when the paper is primarily:**
- A review or survey article
- A book or book chapter

**Exclude when it is primarily:**
- Original research with new experiments, simulations, materials, or applications (even if it has a “review” section) → classify by the main contribution