# LEAP — Lotka-Euler Airway Pathogen model

Code repository for *Long-range seeding drives exponential growth in early respiratory viral infection* (Hvid, Nielsen & Sneppen).

## Overview

LEAP is an analytically tractable within-host model of early respiratory viral infection. It represents infection as a population of expanding foci of infection (IF), each growing as a circular plaque at constant radial speed *v*. Each infected cell seeds *q* new foci at distant sites via long-range transport (airflow or bloodstream). This two-scale process — local expansion plus stochastic long-range seeding — reduces to an age-dependent branching process, for which the exponential growth rate is found by solving the Lotka-Euler equation. The model is parameterized using plaque growth and viral load data for SARS-CoV-2 and influenza A virus (IAV).

## Repository contents

| File | Description |
|---|---|
| `LEAP_analysis_and_figs.ipynb` | Main analysis notebook |
| `LEAP_simulation.ipynb` | Stochastic simulation notebook |
| `viral_titer_in_vitro.ipynb` | In vitro viral titer analysis notebook |

## Notebooks

### `LEAP_analysis_and_figs.ipynb`
The primary notebook. Contains the full analytical framework: the main equation, the Lotka-Euler solver, and all derived quantities. Use this notebook to reproduce graphs from the main-text figures and all reported numerical estimates, including the jump parameter *q* for SARS-CoV-2 and IAV, growth rates, and the effect of IFN signaling.

### `LEAP_simulation.ipynb`
A stochastic simulation of the plaque birth process. Individual foci are tracked over time, with births drawn from a Poisson process proportional to the rate of change of infected area. Supports optional features such as a growth delay *a_g*, an IFN-induced growth cap via *p_n*. Used to produce supplementary figures validating the analytical predictions.

### `viral_titer_in_vitro.ipynb`
Processes one-step growth curve data (viral titer vs. time post-infection) for SARS-CoV-2 WT, Delta, and IAV. Titers are discounted by the factor *e*^(−λ*t*) to identify the time point at which the seed cell contributes most to population growth, giving the delay parameter *a_g*. Used to produce supplementary figure S1.

## Key model parameters

| Parameter | Meaning |
|---|---|
| *v* | Radial expansion speed of a plaque |
| *q* | Mean number of new foci seeded per infected cell |
| *p_n* | Probability a cell triggers IFN and halts plaque growth |
| *a_g* | Delay from seeding to onset of plaque growth |
| *a_n* | Age at which a plaque stops growing (determined by *p_n*) |
| *λ* | Exponential growth rate of the infection |
| *λ_0* | Growth rate in the absence of IFN (*p_n* = 0) |

Fitted values for SARS-CoV-2 WT: *a_g* = 16 h, *p_n* = 10⁻⁴, *λ* = 0.25 h⁻¹, *q* ≈ 0.5 cell⁻¹. For IAV: *a_g* = 6 h, *p_n* = 5×10⁻³, *λ* = 0.25 h⁻¹, *q* ≈ 0.07 cell⁻¹.

## Dependencies

Standard scientific Python stack: `numpy`, `scipy`, `matplotlib`, `tqdm`.
