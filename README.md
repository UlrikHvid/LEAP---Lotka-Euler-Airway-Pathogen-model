# LEAP — Lotka-Euler Airway Pathogen model

Code repository for *Bridging spatial scales in respiratory viral infection* (Hvid, Nielsen & Sneppen).

## Overview

LEAP is an analytically tractable within-host model of early respiratory viral infection. It represents infection as a population of expanding foci of infection, each growing as a circular plaque at constant radial speed *v*. Each infected cell seeds *q* new foci at distant sites via long-range transport (airflow or bloodstream). This two-scale process — local expansion plus stochastic long-range seeding — reduces to an age-dependent branching process, for which the exponential growth rate is found by solving the Lotka-Euler equation. The model is parameterized using plaque growth and viral load data for SARS-CoV-2 and influenza A virus (IAV).

## Repository contents

| File | Description |
|---|---|
| `LEAP_analysis_and_figs.ipynb` | Main analysis notebook |
| `LEAP_simulation.ipynb` | Stochastic simulation notebook |
| `Cellular_Automaton.ipynb` | Cellular automaton model of a single focus (SI Appendix C) |
| `viral_titer_in_vitro.ipynb` | In vitro viral titer analysis notebook |

## Notebooks

### `LEAP_analysis_and_figs.ipynb`
The primary notebook. Contains the full analytical framework: the main equation, the Lotka-Euler solver, and all derived quantities. Use this notebook to reproduce graphs from the main-text figures and all reported numerical estimates, including the jump parameter *q* for SARS-CoV-2 and IAV, growth rates, and the effect of IFN signaling.

### `LEAP_simulation.ipynb`
A stochastic simulation of the plaque birth process. Individual foci are tracked over time, with births drawn from a Poisson process proportional to the rate of change of infected area. Supports optional features such as an eclipse phase *a_g* and an IFN-induced growth cap via *p*. Used to produce supplementary figures validating the analytical predictions.

### `Cellular_Automaton.ipynb`
A lattice model of a single focus of infection, used in SI Appendix C to derive the reproduction kernel *b*(*a*) numerically rather than assuming the piecewise-linear form of the main text. Epithelial cells on a square lattice occupy states T (target), E (eclipse), V (shedding), A (antiviral) and D (dead); a fraction *p*(IFN) of exiting E cells instead release interferon, which accumulates as a global scalar and feeds back on both the T → A rate and *p* itself. Running an ensemble of realizations gives the mean shedding rate *F*(*a*), which is inserted into the Lotka-Euler equation and solved for *q*. Produces figs. S4–S6.

Set `mode` in the run cell to `"basic"` (the parameterization reported in the SI), or to `"no_IFN"`, `"strong_IFN"` or `"step_IFN"` to reproduce the three IFN sensitivity checks.

### `viral_titer_in_vitro.ipynb`
Processes one-step growth curve data (viral titer vs. time post-infection) for SARS-CoV-2 WT, Delta, and IAV. Titers are discounted by the factor *e*^(−λ*t*) to identify the time point at which the seed cell contributes most to population growth, giving the eclipse parameter *a_g*. Used to produce supplementary figure S1. SARS-CoV-2 data from Khandelwal et al., 2021, Frontiers Cell. Inf. Micro. IAV data from Frensing et al., 2016, Appl. Microb. cell phys.

## Key model parameters

| Parameter | Meaning |
|---|---|
| *v* | Radial expansion speed of a plaque |
| *q* | Mean number of new foci seeded per infected cell |
| *p* | Probability a cell secretes IFN and halts plaque growth |
| *a_g* | Eclipse phase: delay from seeding to onset of plaque growth |
| *a_n* | Age at which a plaque stops growing (determined by *p*) |
| *λ* | Exponential growth rate of the infection |
| *λ_0* | Growth rate in the absence of IFN (*p* = 0) |

Fitted values for SARS-CoV-2 WT: *a_g* = 16 h, *p* = 10⁻⁴, *λ* = 0.25 h⁻¹, *q* ≈ 0.5 cell⁻¹. For IAV: *a_g* = 6 h, *p* = 5×10⁻³, *λ* = 0.25 h⁻¹, *q* ≈ 0.07 cell⁻¹. The cellular automaton gives somewhat higher estimates, *q* ≈ 0.7 and *q* ≈ 0.2 respectively.

In the cellular automaton, *p* is not a single number but interpolates with interferon concentration from the first-responder probability `p_0` up to the second-responder probability `p_max`, representing the paracrine amplification loop.

## Notes on the code

A few points where the code and the SI Appendix do not line up exactly, none of which affect the reported results:

- **Dead states.** The SI describes a single dead state D. The code splits it into `D_V` (died from viral shedding) and `D_N` (died after releasing IFN). The two are dynamically identical — both inert — so the split is purely for bookkeeping and visualisation.
- **Grid size.** Table S1 gives L = 500. That is the value used for SARS-CoV-2, but the IAV run uses L = 200. This makes no difference: with IAV parameters the plaque is halted by interferon long before the front reaches the boundary, so the grid could be any size above a few hundred. The notebook warns and stops early if the front ever does reach the edge.
- **SARS-CoV-2 burst split.** The two-phase shedding is normalized so the *total* expected burst is exactly β = 500, which means 4.95 virions are shed during V1 and 495.0 during V2 (Table S1 rounds these to 5 and 500). Virion release is Poisson, so the fractional values are meaningful rather than a rounding artifact.

Two things to be aware of when running:

- **Ensemble truncation and *q*.** `run_ensemble` truncates every run to the length of the shortest one so the histories can be stacked. A single run that hits the grid edge early therefore cuts the Lotka-Euler integral short for the whole ensemble and biases *q* upward. If the edge-hit warning fires, enlarge `L` and re-run rather than accepting the *q* value.
- **Plaque size.** Defined throughout as the number of cells that are neither T nor A, i.e. that have been infected at some point. This is computed once, in `CellularAutomaton.counts()` under the key `plaque`; use that key rather than reconstructing it from individual state counts.

## Dependencies

Standard scientific Python stack: `numpy`, `scipy`, `matplotlib`, `tqdm`, plus `multiprocess` for the parallel ensemble runs in `Cellular_Automaton.ipynb`.
