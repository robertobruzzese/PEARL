# PEARL Pipeline 3 — Molecular Dynamics and Binding-Energy Analysis

This directory contains the molecular-dynamics and energetic-evaluation stage of the PEARL project.

Pipeline 3 extends the structural and sequence-based prioritization developed in Pipeline 2 by testing whether the selected extracellular EGFR–peptide interfaces remain stable over time in an explicit-solvent molecular-dynamics environment and by adding a comparative endpoint-energy estimate.

The main purpose of this pipeline is therefore to move from:

```text
static structural prioritization
        ↓
dynamic interface assessment
        +
comparative energetic evaluation
```

The analyses contained in this directory are computational and do not constitute experimental evidence of peptide binding, inhibition, affinity or biological activity.

---

## Relationship with Pipeline 2

Pipeline 2 identified a compact set of peptide candidates through hotspot-centred miniaturization, CLEAR-inspired optimization, FoldX evaluation and Rosetta FlexPepDock refinement.

The main sequences carried forward into the direct Pipeline 3 peptide comparison are:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Their roles are:

- **F0010** — 11-aa reference peptide extracted from the native EGF sequence and corresponding to chain D residues 38–48 in the source EGF structure;
- **CF06** — Rosetta-supported counterfactual candidate;
- **CF02** — counterfactual candidate showing the strongest short-timescale dynamic-stability metrics in the 1 ns comparison.

Pipeline 3 does not replace the Pipeline 2 structural analyses. It adds a dynamic and energetic computational assessment layer.

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
interface/source-region analysis
        ↓
MM/GBSA-like endpoint comparison
```

---

## Biological system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Interface studied:** chains `B–D`
- **Receptor chain:** `B` (extracellular EGFR)
- **Native ligand chain:** `D` (EGF)
- **Reference short peptide:** `F0010 = IGERCQYRDLK`
- **F0010 structural mapping:** chain D residues 38–48
- **Heavy-atom contact cutoff:** 4.5 Å
- **Persistent-contact threshold:** ≥ 50% of analysed MD frames

The B–D interface corresponds to the extracellular EGF–EGFR interaction, with EGFR chain B and EGF chain D. Pipeline 3 first examines this interface in the full EGF–EGFR complex and then directly simulates the extracted F0010 peptide and selected counterfactual peptides bound to EGFR.

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
input receptor–partner structure
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

The 1 ns trajectories used here should be interpreted as computational prototypes suitable for comparative prioritization, not as fully converged long-timescale molecular-dynamics studies.

---

## Notebooks included

- `07a_EGFR_Dimer_OpenMM_MD_Setup.ipynb`
- `07b_EGFR_Dimer_MD_Contact_Persistence.ipynb`
- `07c_EGFR_MD_Hotspot_and_Interface_Validation.ipynb`
- `07d_Selected_Peptide_MD_Comparison.ipynb`
- `07e_Selected_Peptide_MMGBSA_Endpoint_Comparison.ipynb`

The filenames of `07a` and `07b` are retained for reproducibility, although the biological system analysed is the extracellular EGF–EGFR B–D interaction rather than an EGFR kinase-domain dimer.

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

Notebook `07c` is primarily an integration and analysis stage: it combines the MD-derived contact information from `07b` with FoldX-derived information generated in earlier PEARL stages rather than launching another long simulation.

---

# Notebook descriptions

## `07a_EGFR_Dimer_OpenMM_MD_Setup.ipynb`

This notebook prepares the extracellular EGF–EGFR B–D interface for molecular dynamics using OpenMM.

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

This notebook establishes the physical simulation environment used by the subsequent Pipeline 3 notebooks. It does not by itself provide evidence of interface stability or peptide affinity.

---

## `07b_EGFR_Dimer_MD_Contact_Persistence.ipynb`

This notebook performs and analyses the production MD of the extracellular EGF–EGFR B–D interface.

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

The verified analysis identified:

```text
144 unique B–D contacts observed during MD
78 persistent contacts with persistence ≥ 50%
67 initial contacts retained among the persistent set
initial-contact retention = 0.858974
```

A separate periodic-boundary/continuity diagnostic over all 500 production frames gave essentially identical direct and minimum-image B–D distances:

```text
mean direct distance       2.669888 Å
mean minimum-image distance 2.669888 Å
maximum discrepancy        ~9.3 × 10^-8 Å
PBC continuity             PASS
```

No trajectory reconstruction was required.

The result provides dynamic computational support for a maintained extracellular EGF–EGFR B–D interaction network during the 1 ns prototype simulation. It is not experimental confirmation of biological binding or activity.

---

## `07c_EGFR_MD_Hotspot_and_Interface_Validation.ipynb`

This notebook integrates information from:

```text
static B–D interface
        +
historical FoldX BuildModel-derived design anchors
        +
1 ns MD contact persistence
```

The objective is to examine whether residues previously prioritized by structural/energetic criteria are also represented in the dynamically persistent interface.

The reference short peptide is:

```text
F0010
IGERCQYRDLK
chain D residues 38–48
```

Importantly, in `07c` residues D38–48 are analysed **while they are still part of the full EGF chain D**. This notebook therefore does not constitute an autonomous MD simulation of the extracted F0010 peptide. Direct bound-state MD of the extracted F0010 peptide is performed in `07d`.

Main operations include:

- loading persistent-contact information generated in `07b`;
- loading historical FoldX BuildModel-derived energetic design-anchor information;
- mapping MD persistence onto chain D residues;
- mapping the D38–48 source region of F0010 onto the dynamic interface;
- identifying residues with both FoldX-derived and MD support;
- examining persistent receptor contacts involving the D38–48 source region.

### Main result

For the D38–48 F0010 source region:

```text
10 / 11 positions
are represented in MD-persistent interface contacts
```

and:

```text
6 / 11 positions
have both historical FoldX-derived design-anchor support
and MD-persistent interface support
```

The six positions are:

```text
G39
R41
Q43
Y44
R45
L47
```

The analysis identified:

```text
34 persistent EGFR contacts involving EGF residues D38–48
```

These results provide computational support for persistence of the **source region of F0010 within the full EGF–EGFR complex**. Agreement between FoldX-derived prioritization and MD persistence is computational convergence, not experimental validation.

---

## `07d_Selected_Peptide_MD_Comparison.ipynb`

This notebook performs a direct molecular-dynamics comparison of three selected peptide complexes:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Each candidate is simulated using the same explicit-solvent OpenMM protocol.

The comparison asks whether the optimized counterfactual peptides preserve or improve dynamic receptor–peptide behaviour relative to **F0010, an 11-aa reference peptide extracted from the native EGF sequence**.

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

Under this 1 ns protocol, CF02 has the lowest peptide RMSD and RMSF and the highest mean contact persistence, whereas CF06 maintains the largest number of persistent receptor–peptide contacts.

F0010 remains dynamically associated with the receptor but is more flexible than the two counterfactual candidates over this short production trajectory.

### PBC and molecular-continuity verification

A separate diagnostic was performed on all 500 production frames for F0010, CF06 and CF02. Direct receptor–peptide distances and periodic minimum-image distances agreed to approximately `10^-7 Å` for every candidate:

```text
F0010  direct mean = 2.633036 Å   PBC mean = 2.633036 Å   PASS
CF06   direct mean = 2.677862 Å   PBC mean = 2.677862 Å   PASS
CF02   direct mean = 2.685465 Å   PBC mean = 2.685465 Å   PASS
```

Overall PBC continuity:

```text
True
```

No trajectory reconstruction or repeat MD was required.

The dynamic evidence does not produce exactly the same ordering as the earlier static FoldX/Rosetta analyses. This is interpreted as method-specific information rather than as evidence that one computational metric is universally definitive.

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

The calculation is a **single-trajectory MM/GBSA-like endpoint estimate**. In the verified run, all three candidates were evaluated with the same energetic protocol:

```text
Protein force field      AMBER ff14SB
Implicit-solvent model   implicit/gbn2.xml
Snapshots per candidate  50
```

More negative values are interpreted as more favourable only within this specific comparative protocol.

### Main endpoint results

| Candidate | Mean endpoint energy (kcal/mol) |
|---|---:|
| **CF06** | **−65.66 ± 6.80** |
| **F0010** | −59.46 ± 5.96 |
| **CF02** | −56.62 ± 4.63 |

The `±` values above are the standard deviations across the 50 selected snapshots. These snapshots are **time-correlated frames sampled from a single trajectory for each candidate** and must not be interpreted as 50 independent MD replicates.

Within this common protocol, the endpoint energetic ordering is:

```text
CF06 > F0010 > CF02
```

where `>` denotes a more favourable (more negative) endpoint-energy proxy, not experimentally demonstrated affinity.

The endpoint ordering differs from the short-timescale dynamic-stability picture, in which CF02 shows the lowest RMSD/RMSF and highest mean contact persistence. Different computational observables probe different aspects of receptor–peptide behaviour.

The endpoint values are comparable only when the force field, implicit-solvent model, preparation procedure, atom selection, snapshot strategy and endpoint-energy definition are held consistent. They must not be combined directly with endpoint values produced by a different energetic protocol.

---

## Supplementary 1-to-10 ns continuation notebook

A separate notebook,

```text
07d_OpenMM_continue_from_1ns_to_10ns.ipynb
```

was prepared as an **accessory continuation workflow**. It is not part of the main `07a → 07e` Pipeline 3 execution sequence.

Its purpose is to restart F0010, CF06 and CF02 from their saved 1 ns OpenMM states and, if deliberately executed, extend each trajectory by a further 9 ns to reach 10 ns total.

The notebook does not repeat minimization, NVT or NPT. It is kept separately because it is a continuation/extension workflow rather than a prerequisite for the results reported in the current Pipeline 3 analysis.

The quantitative results documented in this README are based on the verified **1 ns production trajectories**. No claim is made here that the supplementary 1-to-10 ns continuation was executed, nor are any 10 ns results used in the current Pipeline 3 conclusions.

---

# Integrated Pipeline 3 interpretation

Pipeline 3 deliberately avoids selecting a final peptide from one metric alone.

The principal method-specific observations are:

```text
Evidence / metric                       F0010       CF06       CF02
--------------------------------------------------------------------
Reference peptide                         ✓
Counterfactual candidate                              ✓          ✓
Mean peptide RMSD (Å)                   1.966       1.293      0.901
Mean peptide RMSF (Å)                   1.164       0.904      0.737
Persistent contacts                       43          45         43
Mean contact persistence                0.541       0.529      0.603
Endpoint proxy (kcal/mol)             -59.46      -65.66     -56.62
```

Thus, within the current protocols:

- CF02 shows the strongest short-timescale dynamic-stability metrics;
- CF06 has the largest number of persistent contacts and the most favourable 07e endpoint-energy proxy;
- F0010 provides the EGF-derived reference peptide against which the counterfactual candidates are compared.

These observations are complementary computational criteria and should not be interpreted as experimental proof that one peptide has higher biological affinity, inhibitory activity or efficacy.

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
- contact persistence;
- PBC/continuity diagnostics.

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

Typical Python dependencies include Python 3.11, OpenMM, PDBFixer, MDAnalysis, NumPy, pandas, Matplotlib, pathlib and standard Python scientific utilities.

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
- FoldX-derived/MD integration tables;
- comparative peptide MD summaries;
- endpoint-energy snapshot and summary tables;
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

is used first to verify that system preparation, OpenMM context creation, trajectory generation and downstream analysis work correctly.

Final comparative results are generated with the corresponding production configuration, for example:

```python
FAST_TEST_MODE = False
```

Smoke-test results must not be interpreted as final scientific results.

**Do not rerun production MD merely to regenerate documentation or reports when the verified production trajectories and saved analysis outputs already exist.**

---

## Interpretation and limitations

### Short production trajectories

The current production simulations are 1 ns prototypes. They are useful for detecting rapid structural instability, comparing short-timescale peptide flexibility, measuring contact persistence and prioritizing candidates, but they are not sufficient to demonstrate full conformational convergence.

More rigorous studies would require longer trajectories and preferably independent replicas.

### Force-field and protocol dependence

MD and endpoint results depend on force-field choice, solvent model, protonation states, ion placement, starting structures, equilibration protocol, atom selections and sampling strategy.

Reported computational orderings are therefore protocol-dependent.

### Endpoint energy is not absolute binding free energy

The `07e` calculation is deliberately described as:

```text
MM/GBSA-like single-trajectory endpoint energy
```

rather than a rigorous binding free energy.

It does not include configurational entropy, fully independent receptor and peptide relaxation, multiple independent MD replicas, long-timescale convergence, alchemical free-energy transformations or experimental calibration.

The 50 endpoint snapshots per candidate are correlated frames from one trajectory, not independent replicates.

The endpoint values should therefore be used for **relative computational comparison within the same protocol**, not interpreted directly as experimental `ΔG`, `Kd`, `Ki` or `IC50`.

### Computational convergence is not biological validation

Agreement between FoldX-derived criteria, Rosetta, MD, contact persistence and endpoint-energy calculations can strengthen a computational hypothesis, but it does not establish biological activity.

Experimental peptide-binding and inhibition studies would be required for biological validation.

---

## Main Pipeline 3 conclusion

Pipeline 3 adds a dynamic and energetic computational assessment layer to the PEARL peptide-design workflow.

The principal verified findings are:

1. the extracellular EGF–EGFR B–D interface retains a substantial persistent-contact network during the 1 ns production MD, with 78 contacts persistent in at least 50% of frames;
2. 10 of the 11 positions in the D38–48 F0010 source region are represented in MD-persistent interface contacts while that region remains part of full EGF;
3. six positions — G39, R41, Q43, Y44, R45 and L47 — have both historical FoldX-derived design-anchor support and MD-persistent interface support;
4. direct 1 ns MD of the extracted peptide complexes shows the strongest short-timescale dynamic-stability metrics for CF02, whereas CF06 maintains the largest number of persistent contacts;
5. the 07e single-trajectory GBn2 endpoint proxy is most favourable for CF06 among F0010, CF06 and CF02;
6. PBC/continuity diagnostics passed for the full EGF–EGFR production trajectory and for all three direct peptide-comparison trajectories, so no trajectory reconstruction was required;
7. differences among structural, dynamic and energetic criteria are treated as complementary computational evidence rather than forced into a single universal score.

Pipeline 3 therefore supports the PEARL strategy of layered candidate assessment:

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
