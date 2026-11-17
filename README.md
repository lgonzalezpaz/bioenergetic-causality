# bioenergetic-causality

Reference implementation for **"Mitochondrial complex I dysfunction in
idiopathic autism: convergence to a single chronic attractor and causal
resolution by dynamic perturbation."**

A bidirectionally coupled biophysical model of neuronal excitability and
mitochondrial bioenergetics, used to ask whether the long-standing
cause-or-consequence question about mitochondria in autism spectrum disorder
is answerable from steady-state biochemistry — and, where it is not, what
measurement would answer it.

## What the code does

The model has ten state variables split between an **electrical node**
(firing rate, cytosolic Na⁺, cytosolic Ca²⁺) and a **mitochondrial node**
(matrix Ca²⁺, |Δψm|, NADH, ATP, H₂O₂, functional complex I capacity, reduced
scavenging capacity). The two nodes are joined in both directions: firing sets
the ATP and Ca²⁺ load, and matrix redox state feeds back through ATP-sensitive
K⁺ channels, ATP-dependent pump capacity, and ROS-mediated inactivation of
complex I.

A lesion is injected at either node —

* **Mode M** (`lam_M`): fractional loss of complex I capacity — a structural or
  genetic respiratory defect.
* **Mode E** (`lam_E`): fractional excess excitatory drive — an E/I shift.

— and both are titrated to the *same measured complex I activity*, so the
comparison is between two cells that present identically in the clinic.

The analysis then asks what, if anything, still distinguishes them:

| Step | What it computes |
|---|---|
| Calibration | Closed-form, non-fitted: each steady-state condition solved for one scaling constant so the literature control state is exact |
| Continuation | Both lesion branches, with the slow proteostatic loop active and frozen |
| Inverse problem | Joint fit of each mode to three human observables (complex I activity, H₂O₂ emission, mtDNA copy number), with the Ca²⁺→ROS gain estimated rather than assumed |
| Phenotype matching | Titration of both lesions to the measured complex I activity reported in patients |
| Discriminators | Closed-loop gain Λ, relaxation rate Θ, perturbation asymmetry PAI, and the composite BCI |
| Directed flow | Transfer entropy and cross-correlation lead between \|Δψm\| and firing, with noise matched at both nodes |
| Monte Carlo | Held-out classification AUC across jittered parameter draws (train/test split fixes discriminator signs on disjoint data) |
| Closure time | Decay of discriminability with time since lesion onset |
| Sensitivity | Saltelli total-effect indices of the mode gap over ±30 % ranges |

## Install and run

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt          # or: conda env create -f environment.yml
python src/run_all.py                    # ~6 min, one CPU core
```

`--quick` runs a reduced-size version (~1 min) for smoke-testing.
No GPU, cluster or specialised hardware is required. The random seed is fixed
in the driver, so output is bitwise reproducible in an identical environment.

## Layout

```
src/model.py                       model definition, closed-form calibration, lesion injection
src/analysis.py                    continuation, inverse problem, matching, discriminators, transfer entropy
src/run_all.py                     single driver: every figure, table and number
src/redo_figures.py                regenerates all 5 vector PDF and 400-dpi PNG figures
src/redo_mc_sobol.py               recomputes the Monte-Carlo and sensitivity steps alone
results/results.json               complete numerical output
results/table1_parameters.csv      Table 1: model parameters, units, and provenance
results/table2_control_state.csv   Table 2: control steady state vs. literature targets
results/table3_validation.csv      Table 3: published human validation set vs. mode fits
results/table4_discriminators.csv  Table 4: dynamic discriminators at matched points
results/table5_sobol.csv           Table 5: Sobol sensitivity indices (±30% ranges)
figures/figure1..5.{pdf,png}       vector PDF and 400-dpi PNG
equations/equations.tex            all 25 model and discriminator equations in LaTeX
```

Note on tables: **Table 4** presents the dynamic discriminators at the phenotype-matched operating points (`results/table4_discriminators.csv`), and **Table 5** presents the Sobol variance-based sensitivity indices (`results/table5_sobol.csv`).

## Human data

No new human data were collected and no patient-level data were accessed. The
validation set in `analysis.py` (`LQVS`) is a set of published central
estimates with their reported intervals; each entry carries its source string.
See Table 3 of the manuscript.

## Parameters

Every parameter is either taken from the literature — with the source recorded
in `model.PARAM_META` / `model.PARAM_SOURCES` — or fixed by the calibration, which is solved in
closed form rather than fitted. No disease observable enters the calibration,
so the model's behaviour under lesion is a prediction.

## Citation

See `CITATION.cff`. Licensed under MIT (`LICENSE`).
