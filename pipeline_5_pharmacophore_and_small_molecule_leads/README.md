# PEARL Pipeline 5 — Pharmacophore and Small-Molecule Lead Discovery

**PEARL — Peptide Extraction and AI-guided Refinement for Ligand design**  
Target: **EGFR kinase asymmetric dimer, PDB 3NJP**  
Pipeline 5: **MD-derived pharmacophore → small-molecule generation/screening → chemistry filtering → DiffDock → Vina refinement → integrated lead selection**

Repository folder:  
https://github.com/robertobruzzese/PEARL/tree/main/pipeline_5_pharmacophore_and_small_molecule_leads

---

## Overview

Pipeline 5 translates the structural information learned from the peptide–EGFR system into a small-molecule lead-discovery workflow.

The pipeline starts from peptide–protein molecular-dynamics ensembles generated upstream, extracts a consensus pharmacophore, screens a REINVENT4-generated molecular library against that pharmacophore, applies drug-likeness and liability filters, docks chemically acceptable candidates to EGFR with DiffDock, locally re-scores/refines the selected poses with AutoDock Vina, and finally integrates the independent evidence streams into a transparent lead-prioritization scheme.

```text
MD peptide–EGFR ensembles
        │
        ▼
09a  MD-derived pharmacophore extraction
        │
        ▼
09b  Pharmacophore consolidation
        │
        ▼
09c  REINVENT4 molecular sampling
     + pharmacophore screening
        │
        ▼
09d  Drug-likeness / liabilities / diversity
        │
        ▼
09e  DiffDock docking + pose/contact analysis
        │
        ▼
09f  Vina local rescoring / refinement
        │
        ▼
09g  Integrated cross-method lead selection
```

The final result is a **computational prioritization**, not experimental evidence of binding.

---

## Notebook map

| Notebook | Purpose | Main output |
|---|---|---|
| `09a_MD_Derived_Pharmacophore_Extraction*.ipynb` | Extract persistent pharmacophoric features from peptide–EGFR MD ensembles | Persistent HBD/HBA/HYD/ARO/POS/NEG features and consensus pharmacophore |
| `09b_Pharmacophore_Consolidation_and_Screening_Ready_Model*.ipynb` | Consolidate redundant features into a screening-ready model | Mandatory / optional / contextual pharmacophore groups |
| `09c_Pharmacophore_Constrained_Molecular_Generation*.ipynb` | Acquire REINVENT4 molecules, generate conformers and screen against the pharmacophore | Ranked molecular library and strict pharmacophore hits |
| `09d_Chemical_Filtering_Druglikeness_Liabilities_Diversity*.ipynb` | Evaluate drug-likeness, structural alerts, synthetic accessibility and diversity | Chemically eligible docking shortlist |
| `09e_DiffDock_Docking_and_Pose_Interaction_Evaluation*.ipynb` | Dock shortlisted molecules and characterize receptor engagement | Best DiffDock pose per candidate |
| `09f_Vina_Local_Rescoring_and_Interaction_Refinement*.ipynb` | Score the selected DiffDock pose with Vina and locally optimize it | Comparative Vina energetic ranking and local pose RMSD |
| `09g_Integrated_Pipeline5_Lead_Selection*.ipynb` | Integrate pharmacophore, chemistry, DiffDock, contacts and Vina evidence | Final multi-criterion lead priority |

---

## 09a — MD-derived pharmacophore extraction

The first stage extracts pharmacophoric information from the peptide–EGFR MD ensembles.

Three upstream peptide candidates were used:

```text
CF02
CF06
MPNN_NEW_05
```

Feature families include:

```text
HBD  hydrogen-bond donor
HBA  hydrogen-bond acceptor
HYD  hydrophobic
ARO  aromatic
POS  positively charged
NEG  negatively charged
```

### Production result

```text
100 frames / candidate
300 frames total
5330 active feature observations
59 persistent candidate-level features
32 spatial clusters
17 core consensus pharmacophore features
ALL QC PASSED: True
```

The pharmacophore is also exported in an RDKit Pharm3D-compatible representation.

---

## 09b — Pharmacophore consolidation

The 17 core consensus features are spatially consolidated to reduce redundancy and construct a practical screening model.

A redundancy radius of approximately **1.25 Å** is used.

The resulting model contains:

```text
10 consolidated groups
5 mandatory groups
4 optional groups
1 contextual group
```

Screening rule:

```text
all mandatory groups
+
at least one optional group
```

This stage converts the raw MD-derived feature map into a screening-ready molecular hypothesis.

---

## 09c — Molecular generation/acquisition and pharmacophore screening

The production run uses **REINVENT4** as an external molecular-generation backend.

The REINVENT4 run sampled:

```text
5000 requested SMILES
```

The notebook then analyzes a production subset of:

```text
1000 molecules
996 successfully embedded in 3D
```

For each molecule, multiple conformers are generated and aligned against the screening-ready pharmacophore.

### Production result

```text
83 molecules with complete mandatory assignments
2 strict pharmacophore-positive molecules
ALL TECHNICAL QC PASSED: True
```

The two strict hits were:

```text
MOL00570
MOL00336
```

Both showed poor downstream medicinal-chemistry properties and were not promoted as final docking leads.

### Important methodological note

The current implementation uses:

```text
unconstrained REINVENT4 de novo sampling
+
downstream pharmacophore screening
```

It should therefore **not** be described as pharmacophore-conditioned REINVENT4 reinforcement learning.

The pharmacophore acts as a post-generation structural filter in the implemented production workflow.

---

## 09d — Chemical filtering, drug-likeness and diversity

09d evaluates pharmacophore-supported candidates using cheminformatic descriptors and structural-alert filters.

The analysis includes:

- molecular weight;
- cLogP;
- TPSA;
- H-bond donors and acceptors;
- rotatable bonds;
- formal charge;
- ring count;
- QED;
- Fsp3;
- Lipinski-style criteria;
- Veber-style criteria;
- PAINS alerts;
- Brenk alerts;
- RDKit synthetic-accessibility score when available;
- Morgan fingerprints;
- Tanimoto similarity;
- Butina clustering.

A key design decision is that **strict pharmacophore evidence and chemical eligibility remain separate concepts**.

### Strict hits

The two strict 09c pharmacophore hits were chemically poor:

```text
MOL00336  CHEM_FAIL
MOL00570  CHEM_FAIL
```

Thus:

```text
strict pharmacophore hits eligible for primary docking = 0
```

### Rescue track

A chemically acceptable near-miss rescue track was therefore retained.

Final 09d docking shortlist:

```text
MOL00053
MOL00484
MOL00273
MOL00600
MOL00294
MOL00857
MOL00583
MOL00489
```

All eight are:

```text
docking_track        = NEAR_MISS_RESCUE
pharmacophore_status = NEAR_MISS_MANDATORY_ONLY
chemistry_class      = CHEM_PASS
```

This provenance is preserved throughout all downstream notebooks.

---

## 09e — DiffDock docking and interaction evaluation

The eight chemically eligible rescue candidates are docked to **EGFR chain B from PDB 3NJP** using DiffDock.

Production settings:

```text
8 candidates
10 DiffDock samples / candidate
20 inference steps
80 pose files total
```

All eight candidates produced the full set of ten poses.

For every pose, 09e computes descriptive structural metrics including:

```text
DiffDock confidence
minimum receptor–ligand distance
number of contacted receptor residues
ligand heavy-atom contact fraction
severe-clash flag
```

A `pose_support_score` is used only as a descriptive pose-selection criterion; it is not interpreted as a physical binding energy.

### 09e selected-pose ranking

```text
1  MOL00273
2  MOL00583
3  MOL00600
4  MOL00484
5  MOL00857
6  MOL00053
7  MOL00489
8  MOL00294
```

Representative selected-pose values:

```text
MOL00273
DiffDock confidence     -0.88
pose support             0.850
contacted residues       7
ligand contact fraction  0.826

MOL00583
DiffDock confidence     -2.02
pose support             0.827
contacted residues      12
ligand contact fraction  0.933

MOL00484
pose support             0.770
contacted residues      14
ligand contact fraction  1.000
```

No docking result changes the pharmacophore provenance: all eight remain rescue candidates.

---

## 09f — AutoDock Vina local energetic refinement

09f adds a second, independent docking-scoring layer to the top four 09e candidates:

```text
MOL00273
MOL00583
MOL00600
MOL00484
```

The procedure deliberately does **not** perform a new global Vina docking search.

Instead:

```text
selected DiffDock pose
        ↓
Vina score of the unchanged pose
        ↓
short local Vina optimization
        ↓
Vina score after optimization
        ↓
DiffDock → Vina local heavy-atom RMSD
```

### Production result

| Candidate | Vina before | Vina after | Δ local optimization | Local RMSD |
|---|---:|---:|---:|---:|
| MOL00600 | 1.998 | **-5.791** | -7.789 | 0.190 Å |
| MOL00583 | 0.384 | **-4.314** | -4.698 | 0.586 Å |
| MOL00484 | -0.556 | **-4.267** | -3.711 | 0.057 Å |
| MOL00273 | -2.982 | **-3.937** | -0.955 | 0.203 Å |

Interpretation:

- **MOL00600** has the strongest post-refinement Vina support.
- **MOL00484** shows exceptional local geometric stability.
- **MOL00583** provides balanced structural and energetic support.
- **MOL00273** has the strongest original DiffDock ranking but weaker final Vina ranking.

`ALL QC PASSED: True`

### Vina is not a binding free-energy calculation

The Vina values are empirical docking-scoring-function outputs and are used only for comparative ranking within the common protocol.

They must **not** be reported as rigorous ΔG, MM-GBSA or MM-PBSA binding free energies.

---

## 09g — Integrated lead selection

09g closes Pipeline 5 by combining six independent evidence axes:

```text
1. pharmacophore fit
2. chemistry / drug-likeness
3. DiffDock structural support
4. receptor engagement
5. Vina energetic support
6. local geometry stability
```

No raw heterogeneous values are added into a single physical score.

Instead, each candidate is ranked independently on each evidence axis and receives a support flag when it falls in the top two among the four finalists.

The support count is used only for transparent **priority assignment**.

### Final ranking

| 09g rank | Candidate | Priority | Top-2 support | Interpretation |
|---:|---|---|---:|---|
| **1** | **MOL00583** | **TIER 1** | **4 / 6** | Strong cross-method support |
| 2 | MOL00484 | TIER 2 | 3 / 6 | Orthogonal follow-up lead |
| 3 | MOL00600 | TIER 2 | 3 / 6 | Orthogonal follow-up lead |
| 4 | MOL00273 | TIER 2 | 2 / 6 | Method-specific / orthogonal lead |

All four finalists are Pareto non-dominated in the implemented rank-space analysis.

### Primary computational lead: MOL00583

`MOL00583` is selected as the **primary computational lead** because it provides the broadest cross-method convergence.

Evidence-axis ranks:

```text
pharmacophore fit          2
chemistry                  4
DiffDock structural        2
receptor engagement        2
Vina energetic             2
local geometry stability   4
```

Key values:

```text
mandatory pharmacophore RMSD     1.341 Å
DiffDock rank                    2
DiffDock confidence             -2.020
pose-support score               0.827
contacted receptor residues     12
ligand contact fraction          0.933
Vina score after refinement     -4.314
DiffDock → Vina local RMSD       0.586 Å
```

The other finalists remain scientifically useful because they are favored by different methods:

```text
MOL00484  strongest chemistry / receptor-engagement /
          local-geometry support

MOL00600  strongest pharmacophore-fit and Vina energetic support

MOL00273  strongest original DiffDock structural support
```

`ALL QC PASSED: True`

---

## Pipeline 5 funnel

```text
09a
17 core consensus pharmacophore features
        ↓
09b
10 consolidated groups
5 mandatory + 4 optional + 1 contextual
        ↓
09c
1000 molecules analyzed
996 embedded
83 complete mandatory assignments
2 strict pharmacophore hits
        ↓
09d
0 strict hits chemically eligible
8 CHEM_PASS near-miss rescue candidates
        ↓
09e
8 candidates × 10 DiffDock poses
80 poses analyzed
        ↓
09f
top 4 locally re-scored/refined with Vina
        ↓
09g
MOL00583 primary computational lead
MOL00484 / MOL00600 / MOL00273 orthogonal follow-up leads
```

---

## Software and environments

### Main PEARL analysis environment

Typical Python requirements include:

```text
numpy
pandas
matplotlib
scipy
RDKit
MDAnalysis
BioPython
```

Upstream PEARL pipelines additionally use packages such as OpenMM and structural-bioinformatics tools.

### REINVENT4

REINVENT4 was run in a separate environment and its generated SMILES were imported into 09c.

The production sampling configuration used the PubChem prior and requested 5000 unique molecules, with 1000 molecules subsequently analyzed in 09c.

### DiffDock

DiffDock was run in a dedicated environment.

The Apple-Silicon setup used during this project included:

```text
Python 3.10
PyTorch 2.5.1
torch-geometric
torch-cluster
torch-scatter
e3nn
fair-esm
ProDy
RDKit
```

DiffDock was executed on CPU on macOS for compatibility.

The first run may download the ESM2 model and generate SO(2)/SO(3) lookup tables.

### AutoDock Vina / Meeko

A separate Conda environment was used:

```bash
conda create -n pearl-vina python=3.10 -y
conda activate pearl-vina
conda install -c conda-forge vina meeko rdkit pandas numpy scipy gemmi -y
```

For receptor preparation, PDB 3NJP chain-B residue 172 contains two alternate conformations with equal occupancy:

```text
B:172 altloc A = 0.50
B:172 altloc B = 0.50
```

The production protocol deterministically selected:

```text
B:172=A
```

during Meeko receptor preparation.

---

## Suggested execution order

The notebooks should be run sequentially:

```text
09a
 ↓
09b
 ↓
09c
 ↓
09d
 ↓
09e
 ↓
09f
 ↓
09g
```

Several stages depend on files exported by the previous notebook, so the `outputs/` directory should be preserved between runs.

09c, 09e and 09f contain external-tool stages:

```text
09c → REINVENT4 sampling
09e → DiffDock inference
09f → AutoDock Vina / Meeko
```

The corresponding notebook prepares the external input files and then re-imports the generated results for analysis.

---

## Output directories

The main output roots are:

```text
outputs/
├── pipeline_5_pharmacophore_09a_...
├── pipeline_5_pharmacophore_09b_consolidated/
├── pipeline_5_pharmacophore_09c_generation/
├── pipeline_5_pharmacophore_09d_chemical_filtering/
├── pipeline_5_pharmacophore_09e_diffdock/
├── pipeline_5_pharmacophore_09f_vina_refinement/
└── pipeline_5_pharmacophore_09g_integrated_lead_selection/
```

Each notebook writes some combination of:

```text
tables/
plots/
structures/
reports/
inputs/
```

The final integrated table is:

```text
outputs/pipeline_5_pharmacophore_09g_integrated_lead_selection/
tables/09g_integrated_lead_priority.csv
```

The final automatic report is:

```text
outputs/pipeline_5_pharmacophore_09g_integrated_lead_selection/
reports/09g_pipeline5_final_report.md
```

---

## Scientific caveats

Pipeline 5 is a **computational prototype** and its output should be interpreted accordingly.

1. The current REINVENT4 production run is unconstrained molecular sampling followed by pharmacophore screening; it is not pharmacophore-conditioned RL generation.
2. The two strict pharmacophore hits failed downstream chemistry criteria, so the final docking panel consists of chemically acceptable pharmacophore near-misses.
3. DiffDock confidence is not an affinity or free-energy estimate.
4. The 09e `pose_support_score` is a descriptive composite for pose selection, not a physical energy.
5. Vina scores are empirical docking scores and are not rigorous binding free energies.
6. The implemented 09f Vina layer is **not the MM-GBSA calculation originally envisaged in the project proposal**.
7. The final 09g priority is based on cross-method rank convergence rather than an artificial sum of incompatible raw scores.
8. `MOL00583` is therefore a **primary computational lead for follow-up**, not an experimentally validated EGFR inhibitor.
9. Experimental binding, functional and selectivity assays would be required to establish biological activity.

---

## Final result

Pipeline 5 successfully connects MD-derived peptide information to small-molecule prioritization:

```text
peptide MD
→ pharmacophore
→ molecular generation
→ pharmacophore matching
→ medicinal-chemistry filtering
→ DiffDock
→ Vina local refinement
→ multi-criterion lead selection
```

Final computational priority:

```text
1. MOL00583  — primary computational lead
2. MOL00484  — orthogonal follow-up lead
3. MOL00600  — orthogonal follow-up lead
4. MOL00273  — orthogonal follow-up lead
```

The most important conclusion is not that all scoring methods agree, but that the pipeline explicitly preserves their **convergence and disagreement** and uses those differences as part of the final scientific interpretation.

---

## Project

**PEARL — Peptide Extraction and AI-guided Refinement for Ligand design**

Target system:

```text
EGFR kinase asymmetric dimer
PDB: 3NJP
```

This repository contains a research/educational computational workflow. It is not intended for clinical use.
