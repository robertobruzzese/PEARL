
PEARL Pipeline 5 — Pharmacophore, Small-Molecule Lead Discovery and Post-selection Validation

**PEARL — Peptide Extraction and AI-guided Refinement for Ligand design**  
Target: **EGFR extracellular domain, PDB 3NJP chain B**  
Pipeline 5: **MD-derived pharmacophore → small-molecule generation/screening → chemistry filtering → DiffDock → Vina refinement → integrated lead selection → lead MD → endpoint energetics**

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
        │
        ▼
09h  MOL00583–EGFR molecular-dynamics validation
        │
        ▼
09i  MOL00583–EGFR MM/GBSA-like endpoint analysis
```

After selection in 09g, 09h and 09i evaluate the selected lead dynamically and energetically. They do not rerank the four finalists or change their pharmacophore provenance. The final result remains **computational prioritization with preliminary post-selection evidence**, not experimental evidence of binding.

This README consolidates the operational instructions for 09a–09i. Recorded 09h/09i results were verified on 13 September 2026; separate stage READMEs are not required.

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
| `09h_MOL00583_EGFR_MD_Validation*.ipynb` | Validate the 09f pose of the lead selected in 09g using explicit-solvent MD | Trajectories, RMSD/RMSF, retention, contacts, QC and 09i manifest |
| `09i_MOL00583_EGFR_MMGBSA_Endpoint_Analysis*.ipynb` | Evaluate snapshots from the 1 ns 09h run using GBn2/ACE endpoint energetics | Energy components, temporal diagnostics, QC and report |

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

09g completes lead prioritization by combining six evidence axes:

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

## 09h — MOL00583–EGFR molecular-dynamics validation

09h follows **09g** and starts from the **Vina-refined MOL00583 pose exported by 09f**. The receptor is EGFR extracellular chain B from 3NJP. This is not a kinase-domain simulation. The selected molecule remains `TIER_1_PRIMARY_COMPUTATIONAL_LEAD`, `NEAR_MISS_MANDATORY_ONLY` and `NEAR_MISS_RESCUE`.

### Protocol and inputs

The protocol follows Pipeline 3 notebooks 07a/07d: OpenMM, AMBER ff14SB (`amber14/protein.ff14SB.xml`), TIP3P (`amber14/tip3p.xml`), 300 K, 1 bar, 0.15 M added NaCl with neutralization, 1 nm padding/cutoff and a 2 fs timestep. MOL00583 uses OpenFF Sage `openff-2.2.1` with AM1-BCC charges generated through AmberTools. The formal ligand microstate is inherited from the SDF; protein preparation uses pH 7.4. Production has **no positional restraints**.

Required files relative to `PROJECT_ROOT`:

```text
outputs/pipeline_5_pharmacophore_09f_vina_refinement/
  inputs/09f_receptor_chain_B.pdb
  inputs/MOL00583_diffdock_pose_H.sdf
  structures/MOL00583_09f_vina_local_minimized.sdf
  tables/09f_QC.csv
outputs/pipeline_5_pharmacophore_09g_integrated_lead_selection/
  tables/09g_integrated_lead_priority.csv
  tables/09g_QC.csv
```

Discovery checks the current directory and the historical `~/Desktop/venv` root; configure `PROJECT_ROOT` explicitly if needed. No missing inputs are fabricated, and no alternative DiffDock pose or SMILES-generated conformation is substituted. Missing files, failed upstream QC, incompatible identity/tier or pose coordinates cause an explicit error. Altloc A follows 09f's choice for B:172.

Missing atoms and structural discontinuities are recorded in `tables/preparation_qc.json`. Missing loops/glycans are not automatically reconstructed; a PDB without SEQRES cannot establish sequence completeness. If required, review/correct the model and document `STRUCTURE_REVIEW_NOTE`; do not fill the note merely to bypass a check.

### Running in Jupyter

Use **Python (PEARL MD 09h)**. The Jupyter server and notebook kernel can belong to different environments; selecting the correct kernel is essential.

1. With `RUN_MD=False`, run the notebook for input/chemistry checks. The “Preflight completato” stop is intentional.
2. Set `RUN_MD=True`, `FAST_TEST_MODE=True`, restart the kernel and run all cells for the short test.
3. Save a separate executed TEST=True copy. For production, keep `RUN_MD=True` and set `FAST_TEST_MODE=False`, restart and run all cells. Save the executed TEST=False copy too.

| Mode | NVT | NPT | Production | Saved frames | Frames summarized after 20% discard |
|---|---:|---:|---:|---:|---:|
| `FAST_TEST_MODE=True` | 10 ps | 10 ps | 50 ps | 25 | 20 |
| `FAST_TEST_MODE=False` | 100 ps | 100 ps | 1,000 ps | 500 | 400 |

Changing the filename alone does not change the settings. `False` runs a longer simulation; it does not accelerate it. For additional independent replicas change `SEED`; longer studies require changing `PRODUCTION_PS` and evaluating convergence.

### Recorded results

| Metric | 50 ps test | 1 ns production |
|---|---:|---:|
| Technical QC | Passed | Passed |
| Mean ligand heavy-atom RMSD after EGFR Cα fit | 1.90 Å | 2.09 Å |
| Descriptive site-retention fraction, analyzed frames | 100% | 100% |
| Residues with contact occupancy ≥50% | 14 | 11 |

The reference is the prepared initial complex derived from 09f; the ligand is not independently fitted. Contacts use a 4.5 Å heavy-atom cutoff. The initial site is defined within 6 Å; descriptive retention requires site contact and ligand/site geometric-center distance ≤10 Å. RMSF is evaluated for site Cα atoms after receptor alignment.

In the 1 ns run, mean ligand RMSD rises from 1.95 to 2.24 Å between the two analyzed halves; the mean fraction of initial contacts retained falls from approximately 85% to 74%. Three geometric hydrogen-bond candidates have occupancies of only 0.25%, 2.5% and 0.5%, not three persistent hydrogen bonds. The ligand remains in the site by the implemented criterion while interactions reorganize. This does not establish long-term stability, convergence or affinity.

### Outputs, progress and recovery

Each run creates a unique folder under `outputs/pipeline_5_pharmacophore_09h_mol00583_md/` relative to the Jupyter working directory. It contains prepared/solvated/final structures, ligand chemistry with charges, OpenMM systems/states/checkpoints, full and solute DCD trajectories, atom/bond maps, analysis tables and plots, `09h_summary.json`, `09h_report.md`, `provenance.json`, and `09i_manifest.json`.

Recorded runs:

```text
test:       20260912T161426_861450Z_seed20260912
production: 20260912T174721_843455Z_seed20260912
```

Inspect `tables/nvt.csv`, `tables/npt.csv` and `tables/production.csv` without interrupting the kernel. Production logs every 2 ps of simulated time; `Time (ps)` and `Speed (ns/day)` report progress and throughput. Minimization may be silent until `states/minimized.xml` appears. Do not queue diagnostic cells in the occupied kernel or rerun the production cell while it is active.

There is no automatic resume. Interrupted runs retain partial outputs and checkpoints. A new execution should restart the kernel and create a new run. Expert continuation requires compatible checkpoint/system/integrator and separately recorded trajectory segments; do not overwrite or blindly concatenate earlier output.

---

## 09i — MOL00583–EGFR endpoint energetics

09i uses the **1 ns production trajectory from 09h**, without new MD, new docking or recalculation of AM1-BCC charges. Both 09i test and production modes use that same trajectory: they differ only in the number of sampled snapshots.

### Method

Single-trajectory **MM/GBSA-like GBn2/ACE**, NoCutoff, solute/solvent dielectric constants 1/78.5 and kappa=0, following the model selected in Pipeline 3 notebook 07e (`implicit/gbn2.xml`). Kappa=0 is the endpoint model setting, not an implicit reproduction of the 0.15 M salt used during explicit-solvent MD.

The calculation preserves 09h charges, sigma and epsilon. For identical component coordinates in the noncovalent complex, intramolecular MM terms cancel in C−R−L. Direct receptor–ligand Coulomb and Lennard-Jones interactions are therefore evaluated separately and checked against an independent OpenMM C−R−L calculation on the first snapshot. GB polar and ACE contributions are evaluated for complex, receptor and ligand. No snapshot minimization or separate component relaxation is performed.

GBn2 radii and screening parameters are generated from the solute topology and retained across components. Unsupported forces, cross-component covalent terms/exceptions, virtual sites, mismatched maps or broken coordinate continuity cause an explicit error. The saved analysis XML systems contain two GB forces for decomposition: **they are not MD systems and their force-group energies must not be blindly summed**.

Compared with 07e, 09i excludes the first 20% of production and uses the AMBER/OpenFF small-molecule system from 09h. Do not directly compare these values with peptide endpoint results without harmonizing protocols. There is no ranking with a single ligand.

### Inputs and execution

Select **Python (PEARL MD 09h)** in Jupyter. Configure `INPUT_RUN`/`PROJECT_ROOT` if files move; the recorded source run is explicitly named, never chosen as the newest folder:

```text
~/Desktop/venv/outputs/pipeline_5_pharmacophore_09h_mol00583_md/
20260912T174721_843455Z_seed20260912/
```

Required files within that run:

```text
09i_manifest.json
09h_summary.json
provenance.json
tables/09h_QC.csv
tables/solute_atom_map.csv
tables/solute_bonds.json
states/physical_system.xml
structures/solvated.pdb
trajectories/production_solute.dcd
inputs/ligand_openff.json
```

1. `RUN_ENERGY=False`: input preflight with an intentional stop.
2. `RUN_ENERGY=True`, `FAST_TEST_MODE=True`: three snapshots at 202, 600 and 1,000 ps for a technical test.
3. `RUN_ENERGY=True`, `FAST_TEST_MODE=False`: 50 snapshots uniformly distributed over 202–1,000 ps, after excluding the first 20% of production.

Restart the kernel and run all cells after changing settings. Save executed TEST=True/TEST=False copies. Each snapshot prints progress and estimated remaining time and updates its CSV. There is no automatic resume; a fresh execution creates a new folder.

### Recorded results

| Mode | Snapshots | Mean endpoint (kcal/mol) | SD across snapshots (kcal/mol) | Technical QC |
|---|---:|---:|---:|---|
| Test | 3 | −21.25 | 1.90 | Passed |
| Production | 50 | **−20.10** | **2.37** | Passed |

Use the **50-snapshot production result** in the report. The test documents execution, not independent validation. The independent MM check differed by approximately 1.1 × 10⁻⁹ kJ/mol; all requested frames, finite energies and component-sum checks passed.

| Production component | Mean (kcal/mol) |
|---|---:|
| Direct Coulomb | −14.29 |
| van der Waals | −33.96 |
| GB polar solvation | +36.67 |
| ACE surface term | −8.52 |
| Total endpoint proxy | **−20.10** |

The first/second analyzed halves average −21.42/−18.79 kcal/mol. Five consecutive ten-snapshot blocks average −21.21, −21.74, −20.36, −18.67 and −18.53 kcal/mol. Later values are less favorable within this model; convergence is not established. These observations accompany contact reorganization in 09h and do not by themselves demonstrate dissociation.

The SD describes correlated snapshots, not uncertainty of the mean. The estimate omits entropy, independent-component relaxation, replicas and experimental calibration; it is not rigorous binding ΔG, Kd/Ki or evidence of inhibition. A negative value alone does not establish binding.

### Outputs

Unique run folders under `outputs/pipeline_5_pharmacophore_09i_mol00583_endpoint/` contain `09i_summary.json`, `09i_report.md`, `provenance.json`, parameter/system exports, QC and hashes, plus:

```text
tables/endpoint_per_frame_kcal_mol.csv
tables/energy_summary.csv
tables/selected_frames.csv
tables/temporal_blocks.csv
tables/MM_validation.json
tables/09i_QC.csv
plots/09i_endpoint.png
plots/09i_blocks.png
```

Recorded runs:

```text
test:       20260913T104032_858892Z_test
production: 20260913T104616_464776Z_production
```

### Figures for the report

Use production outputs, not test screenshots. `09i_endpoint.png` shows 50 endpoint values and their mean component contributions; `09i_blocks.png` is a supplementary temporal diagnostic, not proof of convergence. The separate ChimeraX figure A–B shows the whole EGFR model and initial/final ligand poses after receptor alignment. The four labeled residues are selected by proximity in the final snapshot, not by contact persistence. Structural rendering does not replace trajectory analysis. A trajectory movie is optional presentation material.

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
        ↓
09h
MOL00583–EGFR MD: 50 ps test and 1 ns production
        ↓
09i
GBn2/ACE endpoint: 3-snapshot test and 50-snapshot analysis
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

Stages 09h–09i use the dedicated environment below, in addition to the upstream structural-bioinformatics tools.

### OpenMM / 09h–09i Jupyter environment

The verified kernel is **Python (PEARL MD 09h)**, Conda environment `pearl-09h`. If it already exists, reuse it. To create it on a compatible Conda platform, run these once in Terminal:

```bash
conda create -n pearl-09h -c conda-forge python=3.11 openmm pdbfixer mdtraj openff-toolkit openff-forcefields openmmforcefields ambertools rdkit numpy pandas matplotlib jupyterlab ipykernel
conda activate pearl-09h
python -m ipykernel install --user --name pearl-09h --display-name "Python (PEARL MD 09h)"
jupyter lab
```

Select that kernel **inside Jupyter**; activating an environment in Terminal alone does not switch an existing notebook. Package resolution is platform-dependent; actual versions and input hashes are recorded per run in `provenance.json`.

If AmberTools is installed but OpenFF cannot find `antechamber`, keep this cell before 09h ligand parameterization:

```python
import os, sys, shutil
from pathlib import Path
kernel_bin = str(Path(sys.executable).parent)
os.environ["PATH"] = kernel_bin + os.pathsep + os.environ.get("PATH", "")
from openff.toolkit.utils import AmberToolsToolkitWrapper
print("Python:", sys.executable)
print("antechamber:", shutil.which("antechamber"))
print("sqm:", shutil.which("sqm"))
print("AmberTools available:", AmberToolsToolkitWrapper.is_available())
```

This corrects executable discovery; it does not install missing software. 09i reuses saved charges and does not invoke AmberTools.

Use `PLATFORM="CPU"` for the verified Mac setup. The OpenCL plugin was present but device initialization returned `No compatible OpenCL platform is available`; changing the platform string alone does not enable a GPU. 09i defaults to `CPU_THREADS=4`. For another machine, test GPU context creation before running a full calculation.

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
 ↓
09h
 ↓
09i
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
├── pipeline_5_pharmacophore_09g_integrated_lead_selection/
├── pipeline_5_pharmacophore_09h_mol00583_md/
│   └── <unique_run>/
└── pipeline_5_pharmacophore_09i_mol00583_endpoint/
    └── <unique_run>/
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

The 09g lead-selection report is:

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
6. 09f remains a Vina scoring/refinement stage. Endpoint MM/GBSA-like analysis is implemented separately in **09i** after 09h; it is not a rigorous binding free-energy calculation.
7. The final 09g priority is based on cross-method rank convergence rather than an artificial sum of incompatible raw scores.
8. `MOL00583` is therefore a **primary computational lead for follow-up**, not an experimentally validated EGFR inhibitor.
9. 09h retention and 09i negative endpoint values are preliminary, model-dependent evidence. One 1 ns trajectory does not establish convergence; the observed temporal changes must be reported.
10. Experimental binding, functional and selectivity assays would be required to establish biological activity.

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
→ selected-lead MD validation
→ endpoint energetic analysis
```

The 09g computational priority is preserved after 09h–09i (no comparative reranking was performed):

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
EGFR extracellular domain
PDB: 3NJP, receptor chain B
Upstream peptide source: EGF chain D interacting with EGFR chain B
```

This repository contains a research/educational computational workflow. It is not intended for clinical use.
