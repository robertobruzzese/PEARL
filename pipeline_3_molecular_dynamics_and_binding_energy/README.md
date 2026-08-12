# PEARL Pipeline 3 — Molecular Dynamics and Binding-Energy Analysis

This directory contains the molecular-dynamics and energetic-validation stage of the PEARL project.

Pipeline 3 extends the structural and sequence-based prioritization developed in Pipeline 2 by testing whether the selected EGFR–peptide interfaces remain stable over time in an explicit-solvent molecular-dynamics environment and by adding a comparative endpoint-energy estimate.

The main purpose of this pipeline is therefore to move from:

```text
static structural prioritization
```

to:

```text
dynamic interface validation
        +
comparative energetic evaluation
```

The analyses contained in this directory are computational and do not constitute experimental evidence of peptide binding, inhibition, affinity or biological activity.

---

## Relationship with Pipeline 2

Pipeline 2 identified a compact set of peptide candidates through hotspot-centred miniaturization, CLEAR-inspired optimization, FoldX evaluation and Rosetta FlexPepDock refinement.

The main sequences carried forward into Pipeline 3 are:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Their roles are:

- **F0010** — natural 11-aa reference peptide derived from the EGFR B–D interface;
- **CF06** — strongest integrated Rosetta/CLEAR counterfactual candidate;
- **CF02** — strongest FoldX/oracle counterfactual candidate.

Pipeline 3 does not replace the Pipeline 2 structural analyses. It adds an independent dynamic validation layer.

```text
Pipeline 2
FoldX + Rosetta + CLEAR
        ↓
selected peptide complexes
        ↓
Pipeline 3
explicit-solvent MD
        ↓
RMSD / RMSF / contact persistence
        ↓
dynamic hotspot validation
        ↓
MM/GBSA-like endpoint comparison
```

---

## Biological system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Interface studied:** chains `B–D`
- **Receptor chain:** `B`
- **Partner / peptide chain:** `D`
- **Reference short peptide:** `F0010 = IGERCQYRDLK`
- **F0010 structural mapping:** chain D residues 38–48
- **Heavy-atom contact cutoff:** 4.5 Å
- **Persistent-contact threshold:** ≥ 50% of analysed MD frames

The B–D interface was identified and structurally characterized in the earlier PEARL pipelines. Pipeline 3 tests whether this interface and the derived peptide contacts persist dynamically.

---

## Molecular-dynamics protocol

The production simulations use OpenMM with an AMBER-family all-atom protein force field and explicit solvent.

Main settings:

```text
Force field        AMBER ff14SB
Water model        TIP3P
Salt               0.15 M NaCl
Temperature        300 K
Pressure           1 bar
Time step          2 fs
Production length  1 ns
Production frames  500
```

The general preparation protocol is:

```text
input receptor–peptide structure
        ↓
structural repair
        ↓
hydrogen addition
        ↓
AMBER ff14SB parameterization
        ↓
TIP3P solvation
        ↓
NaCl addition
        ↓
energy minimization
        ↓
NVT equilibration
        ↓
NPT equilibration
        ↓
production molecular dynamics
        ↓
trajectory analysis
```

The 1 ns trajectories used here should be interpreted as a computational prototype suitable for comparative prioritization, not as a fully converged long-timescale molecular-dynamics study.

---

## Notebooks included

- `07a_EGFR_Dimer_OpenMM_MD_Setup.ipynb`
- `07b_EGFR_Dimer_MD_Contact_Persistence.ipynb`
- `07c_EGFR_MD_Hotspot_and_Interface_Validation.ipynb`
- `07d_Selected_Peptide_MD_Comparison.ipynb`
- `07e_Selected_Peptide_MMGBSA_Endpoint_Comparison.ipynb`

---

## Recommended execution order

```text
07a_EGFR_Dimer_OpenMM_MD_Setup
        ↓
07b_EGFR_Dimer_MD_Contact_Persistence
        ↓
07c_EGFR_MD_Hotspot_and_Interface_Validation
        ↓
07d_Selected_Peptide_MD_Comparison
        ↓
07e_Selected_Peptide_MMGBSA_Endpoint_Comparison
```

Notebook `07c` is primarily an integration and validation stage: it combines the MD-derived contact information from `07b` with hotspot information generated in earlier PEARL pipelines rather than launching another long simulation.

---

# Notebook descriptions

## `07a_EGFR_Dimer_OpenMM_MD_Setup.ipynb`

This notebook prepares the EGFR B–D dimer interface for molecular dynamics using OpenMM.

Main operations include:

- loading PDB structure `3NJP`;
- selecting chains `B` and `D`;
- repairing the protein structure where required;
- removing non-required heterogens;
- adding hydrogens;
- applying the AMBER ff14SB force field;
- solvating the complex with TIP3P water;
- adding ions;
- defining temperature and pressure control;
- energy minimization;
- short NVT and NPT equilibration;
- checking system quality before production MD;
- exporting a production-ready state.

This notebook establishes the physical simulation environment used by the subsequent Pipeline 3 notebooks.

It does not by itself provide evidence of interface stability or peptide affinity.

---

## `07b_EGFR_Dimer_MD_Contact_Persistence.ipynb`

This notebook performs and analyses the production MD of the EGFR B–D interface.

The production prototype consists of a 1 ns trajectory sampled into 500 protein-only frames.

Main analyses include:

- receptor-aligned trajectory processing;
- Cα RMSD analysis;
- residue-level RMSF analysis;
- B–D heavy-atom contact detection using a 4.5 Å cutoff;
- contact-count time series;
- contact persistence over the production trajectory;
- identification of contacts present in at least 50% of frames;
- comparison between initial structural contacts and dynamically persistent contacts;
- chain-specific residue contact-persistence summaries.

### Main result

The analysis identified:

```text
147 unique B–D contacts observed during MD
80 persistent contacts with persistence ≥ 50%
```

The result supports the presence of a dynamically maintained B–D interaction network during the 1 ns prototype simulation.

This should be interpreted as dynamic computational support for the selected interface, not as experimental confirmation of the biological dimer interface.

---

## `07c_EGFR_MD_Hotspot_and_Interface_Validation.ipynb`

This notebook integrates three sources of information:

```text
static B–D interface
        +
FoldX energetic hotspots
        +
1 ns MD contact persistence
```

The objective is to determine which residues are supported simultaneously by structural, energetic and dynamic evidence.

The reference short peptide is:

```text
F0010
IGERCQYRDLK
chain D residues 38–48
```

Main operations include:

- loading the persistent-contact information generated in `07b`;
- loading FoldX energetic-hotspot information from the earlier PEARL stages;
- mapping MD persistence onto chain D residues;
- mapping F0010 onto the dynamic interface;
- identifying residues supported by both FoldX and MD;
- examining persistent receptor contacts involving the F0010 region.

### Main result

For F0010:

```text
11 / 11 peptide positions
are supported by MD-persistent interface contacts
```

and:

```text
6 / 11 positions
are simultaneously supported as FoldX hotspots
and MD-persistent interface residues
```

The six FoldX + MD-supported F0010 positions are associated with:

```text
G39
R41
Q43
Y44
R45
L47
```

The analysis also identified:

```text
32 persistent B–F0010 contacts
```

This provides cross-method computational support for the use of F0010 as a dynamically relevant short interface-derived reference peptide.

---

## `07d_Selected_Peptide_MD_Comparison.ipynb`

This notebook performs a direct molecular-dynamics comparison of three selected peptide complexes:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Each candidate is simulated using the same explicit-solvent OpenMM protocol.

The comparison asks:

> Do the optimized counterfactual peptides preserve or improve the dynamic receptor–peptide interface relative to the natural F0010 reference?

Main analyses include:

- receptor-aligned peptide Cα RMSD;
- peptide Cα RMSF;
- B–peptide heavy-atom contact persistence;
- number of persistent contacts ≥ 50%;
- mean contact persistence;
- comparison of candidate-specific and shared receptor contacts;
- interpretation of mutation-site contacts.

### Main MD results

| Candidate | Mean peptide RMSD (Å) | Mean peptide RMSF (Å) | Persistent contacts ≥50% | Mean contact persistence |
|---|---:|---:|---:|---:|
| **CF02** | **0.901** | **0.737** | 43 | **0.603** |
| **CF06** | 1.293 | 0.904 | **45** | 0.529 |
| **F0010** | 1.966 | 1.164 | 43 | 0.541 |

### Interpretation

`CF02` shows the lowest peptide RMSD, the lowest RMSF and the highest mean contact persistence.

`CF06` maintains the largest number of persistent receptor–peptide contacts.

`F0010` remains dynamically associated with the receptor but is more flexible than the two optimized counterfactual candidates under this 1 ns protocol.

The dynamic evidence therefore does not produce exactly the same ranking as the static FoldX/Rosetta analyses, which is an important result of the layered PEARL validation strategy.

---

## `07e_Selected_Peptide_MMGBSA_Endpoint_Comparison.ipynb`

This notebook adds an energetic comparison using snapshots from the 1 ns trajectories generated in `07d`.

For each selected snapshot:

```text
ΔE_endpoint =
    E_complex
  - E_receptor
  - E_peptide
```

The three terms are evaluated using the same snapshot geometry.

The calculation is a **single-trajectory MM/GBSA-like endpoint estimate** using an AMBER-based molecular-mechanics model with implicit-solvent treatment.

More negative values are interpreted as more favourable only within this specific comparative protocol.

### Main endpoint results

| Candidate | Mean endpoint energy (kcal/mol) |
|---|---:|
| **CF06** | **−65.66 ± 6.80** |
| **F0010** | −59.46 ± 5.96 |
| **CF02** | −56.62 ± 4.63 |

### Interpretation

The endpoint energetic ranking differs from the pure dynamic-stability ranking:

```text
Endpoint energetic proxy:
CF06 > F0010 > CF02
```

whereas the peptide RMSD/RMSF and mean contact-persistence metrics favour `CF02`.

This disagreement is informative rather than contradictory: different computational methods probe different components of receptor–peptide behaviour.

Within Pipeline 3, `CF06` emerges as the strongest energetic candidate, while `CF02` emerges as the strongest dynamic-stability candidate.

---

# Integrated Pipeline 3 interpretation

Pipeline 3 deliberately avoids selecting a final peptide from one metric alone.

The evidence can be summarized as:

```text
                    F0010       CF06        CF02
-----------------------------------------------------
Natural reference     ✓
CLEAR optimized                   ✓           ✓
Rosetta support                  strong
MD RMSD/RMSF                                   best
Persistent contacts              best count
Mean persistence                               best
Endpoint energy                  best
```

The main conclusion is therefore one of **cross-method complementarity**.

`CF06` is particularly important because it combines strong previous structural validation with the most favourable endpoint energetic estimate.

`CF02` is particularly important because it displays the strongest dynamic stability during the 1 ns MD comparison.

The results support carrying more than one peptide lead into later stages rather than treating any single score as definitive.

---

## External software

### OpenMM

OpenMM is used for:

- force-field construction;
- explicit-solvent system preparation;
- minimization;
- NVT/NPT equilibration;
- molecular-dynamics propagation;
- trajectory generation;
- endpoint energy evaluation.

### PDBFixer

PDBFixer is used where required for structural preparation and correction of input PDB structures before simulation.

### MDAnalysis

MDAnalysis is used for trajectory analysis, including:

- structural alignment;
- RMSD;
- RMSF;
- atom selections;
- distance-based contact analysis;
- contact persistence.

---

## Python environment

A dedicated conda environment is recommended.

Example:

```bash
conda create -n pearl-md python=3.11 -y
conda activate pearl-md

conda install -c conda-forge \
    openmm \
    pdbfixer \
    numpy \
    pandas \
    matplotlib \
    jupyter \
    ipykernel \
    mdanalysis \
    -y
```

The notebooks were developed using the Jupyter kernel:

```text
Python (PEARL MD)
```

Typical Python dependencies include:

- Python 3.11;
- OpenMM;
- PDBFixer;
- MDAnalysis;
- NumPy;
- pandas;
- Matplotlib;
- pathlib;
- standard Python scientific utilities.

---

## Expected outputs

Depending on the notebook and run mode, Pipeline 3 may generate:

- repaired and solvated OpenMM systems;
- serialized OpenMM system/state files;
- PDB topology files;
- DCD molecular-dynamics trajectories;
- thermodynamic-state logs;
- protein-only trajectories;
- RMSD time series;
- RMSF tables;
- receptor–partner contact tables;
- contact-persistence matrices;
- persistent-contact subsets;
- hotspot/MD integration tables;
- comparative peptide MD summaries;
- endpoint-energy snapshot tables;
- endpoint-energy summary tables;
- diagnostic plots;
- Markdown reports;
- CSV result files.

Generated data are generally stored under:

```text
outputs/
```

with notebook-specific subdirectories.

Large DCD trajectories, serialized simulation states and other high-volume temporary data should normally not be committed to GitHub unless intentionally archived.

---

## Test mode and production mode

Several MD notebooks use a test/production switch.

Typical logic:

```python
FAST_TEST_MODE = True
```

is used first to verify that:

- system preparation succeeds;
- OpenMM can create a simulation context;
- trajectories are generated;
- analysis code completes;
- no obvious structural failure occurs.

The final comparative analyses should then be rerun with:

```python
FAST_TEST_MODE = False
```

to generate the full 1 ns production trajectories and associated analysis.

Smoke-test results must not be interpreted as final scientific results.

---

## Interpretation and limitations

Pipeline 3 provides a stronger physical validation layer than static scoring alone, but several limitations remain.

### Short production trajectories

The current production simulations are 1 ns prototypes.

They are useful for:

- detecting rapid structural instability;
- comparing short-timescale peptide flexibility;
- measuring contact persistence;
- prioritizing candidates.

They are not sufficient to demonstrate full conformational convergence.

More rigorous studies would require longer trajectories and preferably independent replicas.

### Force-field dependence

MD results depend on:

- force-field choice;
- solvent model;
- protonation states;
- ion placement;
- starting structures;
- equilibration protocol.

The reported ranking is therefore protocol-dependent.

### Endpoint energy is not absolute binding free energy

The `07e` calculation is deliberately described as:

```text
MM/GBSA-like single-trajectory endpoint energy
```

rather than a rigorous binding free energy.

It does not include:

- configurational entropy;
- fully independent receptor and peptide relaxation;
- multiple independent MD replicas;
- long-timescale convergence;
- alchemical free-energy transformations;
- experimental calibration.

The endpoint values should therefore be used for **relative computational prioritization**, not interpreted directly as experimental `ΔG`, `Kd`, `Ki` or `IC50`.

### Computational convergence is not biological validation

Agreement between:

```text
FoldX
Rosetta
MD
contact persistence
endpoint energy
```

strengthens a computational hypothesis but does not establish biological activity.

Experimental peptide-binding and inhibition studies would be required for biological validation.

---

## Main Pipeline 3 conclusion

Pipeline 3 successfully adds a dynamic and energetic validation layer to the PEARL peptide-design workflow.

The principal findings are:

1. the EGFR B–D interface retains a substantial persistent-contact network during the 1 ns production MD;
2. all 11 F0010 positions remain represented in the MD-persistent interface;
3. six F0010 positions are supported simultaneously by FoldX hotspot analysis and MD persistence;
4. both CLEAR-derived candidates are dynamically more rigid than the natural F0010 reference under the current protocol;
5. `CF02` shows the strongest MD stability metrics;
6. `CF06` shows the most favourable MM/GBSA-like endpoint-energy estimate;
7. static structural ranking, dynamic stability and endpoint energy provide complementary rather than identical evidence.

Pipeline 3 therefore supports the PEARL strategy of **layered candidate validation**:

```text
sequence design
        ↓
structural scoring
        ↓
high-resolution refinement
        ↓
molecular dynamics
        ↓
comparative energetic evaluation
        ↓
candidate prioritization
```

The resulting peptide candidates remain computational hypotheses for subsequent AI-guided design, more rigorous free-energy analysis and eventual experimental validation.
