PEARL Pipeline 5 --- Pharmacophore, Small-Molecule Lead Discovery and
Post-selection Validation

**PEARL --- Peptide Extraction and AI-guided Refinement for Ligand
design**\
Target: **EGFR extracellular domain, PDB 3NJP chain B**\
Pipeline 5: **MD-derived pharmacophore → small-molecule
generation/screening → chemistry filtering → DiffDock → Vina refinement
→ integrated lead selection → lead MD → endpoint energetics →
Windows/CUDA free-ligand and bound-complex follow-up simulations**

------------------------------------------------------------------------

## Overview

Pipeline 5 translates the structural information learned from the
peptide--EGFR system into a small-molecule lead-discovery workflow.

The pipeline starts from peptide--protein molecular-dynamics ensembles
generated upstream, extracts a consensus pharmacophore, screens a
REINVENT4-generated molecular library against that pharmacophore,
applies drug-likeness and liability filters, docks chemically acceptable
candidates to EGFR with DiffDock, locally re-scores/refines the selected
poses with AutoDock Vina, and finally integrates multiple partially
dependent computational criteria into a transparent historical
lead-prioritization scheme.

``` text
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
09h  MOL00583–EGFR molecular-dynamics follow-up
        │
        ▼
09i  MOL00583–EGFR MM/GBSA-like endpoint analysis
        │
        ├──────────────► 09j  MOL00583 free in water
        │                    Windows/CUDA, replicate MD
        │
        └──────────────► 09k  MOL00583–EGFR complex replicas
                             Windows/CUDA, RTX 4090
```

After selection in 09g, 09h and 09i evaluate the selected lead
dynamically and energetically. The Windows/CUDA follow-up stages 09j and
09k extend this post-selection validation by comparing the neutral
MOL00583 microstate in free water with longer replicated simulations of
the bound MOL00583--EGFR complex. These stages do not rerank the four
finalists or change their pharmacophore provenance. The final result
remains **computational prioritization with preliminary post-selection
evidence**, not experimental evidence of binding.

This README consolidates the operational instructions for 09a--09k.
Recorded 09h/09i results were verified on 13 September 2026. The
Windows/CUDA extensions 09j and 09k are documented here together with
their stage-specific execution notes; production CUDA status must be
interpreted exactly as reported in the corresponding notebook outputs
and stage notes.

------------------------------------------------------------------------

## Notebook map

  ----------------------------------------------------------------------------------------------------------------------
  Notebook                                                             Purpose                 Main output
  -------------------------------------------------------------------- ----------------------- -------------------------
  `09a_MD_Derived_Pharmacophore_Extraction*.ipynb`                     Extract persistent      Persistent
                                                                       pharmacophoric features HBD/HBA/HYD/ARO/POS/NEG
                                                                       from peptide--EGFR MD   features and consensus
                                                                       ensembles               pharmacophore

  `09b_Pharmacophore_Consolidation_and_Screening_Ready_Model*.ipynb`   Consolidate redundant   Mandatory / optional /
                                                                       features into a         contextual pharmacophore
                                                                       screening-ready model   groups

  `09c_Pharmacophore_Constrained_Molecular_Generation*.ipynb`          Acquire REINVENT4       Ranked molecular library
                                                                       molecules, generate     and strict pharmacophore
                                                                       conformers and screen   hits
                                                                       against the             
                                                                       pharmacophore           

  `09d_Chemical_Filtering_Druglikeness_Liabilities_Diversity*.ipynb`   Evaluate drug-likeness, Chemically eligible
                                                                       structural alerts,      docking shortlist
                                                                       synthetic accessibility 
                                                                       and diversity           

  `09e_DiffDock_Docking_and_Pose_Interaction_Evaluation*.ipynb`        Dock shortlisted        Best DiffDock pose per
                                                                       molecules and           candidate
                                                                       characterize receptor   
                                                                       engagement              

  `09f_Vina_Local_Rescoring_and_Interaction_Refinement*.ipynb`         Score the selected      Comparative Vina
                                                                       DiffDock pose with Vina energetic ranking and
                                                                       and locally optimize it local pose RMSD

  `09g_Integrated_Pipeline5_Lead_Selection*.ipynb`                     Integrate               Final multi-criterion
                                                                       pharmacophore,          lead priority
                                                                       chemistry, DiffDock,    
                                                                       contacts and Vina       
                                                                       evidence                

  `09h_MOL00583_EGFR_MD_Validation*.ipynb`                             Validate the 09f pose   Trajectories, RMSD/RMSF,
                                                                       of the lead selected in retention, contacts, QC
                                                                       09g using               and 09i manifest
                                                                       explicit-solvent MD     

  `09i_MOL00583_EGFR_MMGBSA_Endpoint_Analysis*.ipynb`                  Evaluate snapshots from Energy components,
                                                                       the 1 ns 09h run using  temporal diagnostics, QC
                                                                       GBn2/ACE endpoint       and report
                                                                       energetics              

  `09j_MOL00583_Free_Water_MD_Comparison*.ipynb`                       Simulate the neutral    Replicate trajectories,
                                                                       MOL00583 microstate     ligand RMSD/radius
                                                                       free in water on        diagnostics, hydration
                                                                       Windows/CUDA for        counts, QC and summary
                                                                       conformational and      
                                                                       hydration comparison    
                                                                       with the bound state    

  `09k_MOL00583_EGFR_Bound_Replicates*.ipynb`                          Run longer replicated   Three 10 ns bound
                                                                       MD of the bound         replicas,
                                                                       MOL00583--EGFR complex  contact/RMSD/RMSF
                                                                       on Windows/CUDA / RTX   diagnostics, QC and
                                                                       4090                    comparison-ready outputs
  ----------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 09a --- MD-derived pharmacophore extraction

The first stage extracts pharmacophoric information from the
peptide--EGFR MD ensembles.

Three upstream peptide candidates were used:

``` text
CF02
CF06
MPNN_NEW_05
```

Feature families include:

``` text
HBD  hydrogen-bond donor
HBA  hydrogen-bond acceptor
HYD  hydrophobic
ARO  aromatic
POS  positively charged
NEG  negatively charged
```

### Production result

``` text
100 frames / candidate
300 frames total
5330 active feature observations
59 persistent candidate-level features
32 spatial clusters
17 core consensus pharmacophore features
ALL QC PASSED: True
```

The pharmacophore is also exported in an RDKit Pharm3D-compatible
representation.

------------------------------------------------------------------------

## 09b --- Pharmacophore consolidation

The 17 core consensus features are spatially consolidated to reduce
redundancy and construct a practical screening model.

A redundancy radius of approximately **1.25 Å** is used.

The resulting model contains:

``` text
10 consolidated groups
5 mandatory groups
4 optional groups
1 contextual group
```

Screening rule:

``` text
all mandatory groups
+
at least one optional group
```

This stage converts the raw MD-derived feature map into a
screening-ready molecular hypothesis.

------------------------------------------------------------------------

## 09c --- Molecular generation/acquisition and pharmacophore screening

The production run uses **REINVENT4** as an external
molecular-generation backend.

The REINVENT4 run sampled:

``` text
5000 requested SMILES
```

The notebook then analyzes a production subset of:

``` text
1000 molecules
996 successfully embedded in 3D
```

For each molecule, multiple conformers are generated and aligned against
the screening-ready pharmacophore.

### Production result

``` text
83 molecules with complete mandatory assignments
2 strict pharmacophore-positive molecules
ALL TECHNICAL QC PASSED: True
```

The two strict hits were:

``` text
MOL00570
MOL00336
```

Both showed poor downstream medicinal-chemistry properties and were not
promoted as final docking leads.

### Important methodological note

The current implementation uses:

``` text
unconstrained REINVENT4 de novo sampling
+
downstream pharmacophore screening
```

It should therefore **not** be described as pharmacophore-conditioned
REINVENT4 reinforcement learning.

The pharmacophore acts as a post-generation structural filter in the
implemented production workflow.

------------------------------------------------------------------------

## 09d --- Chemical filtering, drug-likeness and diversity

09d evaluates pharmacophore-supported candidates using cheminformatic
descriptors and structural-alert filters.

The analysis includes:

-   molecular weight;
-   cLogP;
-   TPSA;
-   H-bond donors and acceptors;
-   rotatable bonds;
-   formal charge;
-   ring count;
-   QED;
-   Fsp3;
-   Lipinski-style criteria;
-   Veber-style criteria;
-   PAINS alerts;
-   Brenk alerts;
-   RDKit synthetic-accessibility score when available;
-   Morgan fingerprints;
-   Tanimoto similarity;
-   Butina clustering.

A key design decision is that **strict pharmacophore evidence and
chemical eligibility remain separate concepts**.

### Strict hits

The two strict 09c pharmacophore hits were chemically poor:

``` text
MOL00336  CHEM_FAIL
MOL00570  CHEM_FAIL
```

Thus:

``` text
strict pharmacophore hits eligible for primary docking = 0
```

### Rescue track

A chemically acceptable near-miss rescue track was therefore retained.

Final 09d docking shortlist:

``` text
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

``` text
docking_track        = NEAR_MISS_RESCUE
pharmacophore_status = NEAR_MISS_MANDATORY_ONLY
chemistry_class      = CHEM_PASS
```

This provenance is preserved throughout all downstream notebooks.

------------------------------------------------------------------------

## 09e --- DiffDock docking and interaction evaluation

The eight chemically eligible rescue candidates were docked to **EGFR
chain B from PDB 3NJP** using DiffDock (10 poses per molecule; 80 poses
total). The historical 09e procedure selected one pose per molecule
using DiffDock confidence and descriptive receptor-contact support.

### Historical selected-pose ranking

``` text
1 MOL00273
2 MOL00583
3 MOL00600
4 MOL00484
5 MOL00857
6 MOL00053
7 MOL00489
8 MOL00294
```

This is a **whole-chain/full-surface docking ranking**, not an
EGF-interface-specific ranking. The `pose_support_score` is descriptive
and is not a physical binding energy.

### A24 --- native EGF-interface relevance

A24 compared the historical selected poses with the 43 EGFR residues of
the native EGF--EGFR interface. The selected poses of `MOL00273`,
`MOL00583`, `MOL00600`, `MOL00053`, `MOL00489` and `MOL00294` had no
native-interface overlap under the implemented criterion. `MOL00484` and
`MOL00857` were site-relevant, with 5 and 6 overlapping interface
residues respectively.

Alternative site-relevant poses existed for several molecules, often
with weaker original docking support; `MOL00600` had none among its ten
samples. These alternatives were not automatically substituted for the
historical selected poses.

Therefore:

``` text
historical whole-chain 09e ranking ≠ EGF-interface-specific ranking
```

All eight molecules retain their original near-miss rescue provenance.

------------------------------------------------------------------------

## 09f --- AutoDock Vina local energetic refinement

09f applies AutoDock Vina to the four historical 09e finalists:
`MOL00273`, `MOL00583`, `MOL00600` and `MOL00484`.

It is **local rescoring/refinement of the selected DiffDock poses**, not
a second independent global docking campaign.

  Candidate     Vina before   Vina after   Δ local optimization
  ----------- ------------- ------------ ----------------------
  MOL00600            1.998   **-5.791**                 -7.789
  MOL00583            0.384   **-4.314**                 -4.698
  MOL00484           -0.556   **-4.267**                 -3.711
  MOL00273           -2.982   **-3.937**                 -0.955

### A25 --- receptor-frame displacement

The historical ligand-only `GetBestRMS` quantity must not be interpreted
as receptor-frame pose stability. A25 measured displacement in the
receptor frame from the already generated structures:

  Candidate     receptor-frame RMSD (Å)   centroid displacement (Å)
  ----------- ------------------------- ---------------------------
  MOL00600                        1.494                       0.384
  MOL00583                        1.418                       0.523
  MOL00484                        1.272                       0.612
  MOL00273                        1.118                       0.787

These values support only **limited local rearrangement during
refinement**. They do not establish binding-site stability. In
particular, A24 had already shown that the historical MOL00583 pose
refined here is off-site relative to the native EGF interface. Among
these four original inputs, `MOL00484` retains native-interface
relevance.

Vina scores remain empirical within-protocol scores; they are not
rigorous binding free energies.

------------------------------------------------------------------------

## 09g --- Integrated lead selection

09g historically combined six computational criteria: pharmacophore fit,
chemistry/drug-likeness, DiffDock structural support, receptor
engagement, Vina energetic support and a local-geometry metric.

These are **not six independent evidence streams**. Several derive from
the same candidate structures, docking poses or related scoring chain.
The top-2 support count is retained as provenance of the implemented
prioritization rule.

### Historical 09g ranking

    Historical rank Candidate   Historical label     Top-2 support
  ----------------- ----------- ------------------ ---------------
                  1 MOL00583    TIER 1                       4 / 6
                  2 MOL00484    TIER 2                       3 / 6
                  3 MOL00600    TIER 2                       3 / 6
                  4 MOL00273    TIER 2                       2 / 6

### A26 --- revised evidence interpretation

`MOL00583` remains the **historical computational lead selected by
09g**, but its propagated 09e/09f pose is off-site and requires
site-relevant structural reassessment. `MOL00484` retains an
EGF-interface-relevant selected pose and therefore retains site-relevant
structural support for follow-up. `MOL00600` and `MOL00273` have
off-site historical selected poses.

A26 does **not** automatically rerank the finalists or promote MOL00484
to a new primary lead. It separates the historical ranking from the
later native-interface audit.

------------------------------------------------------------------------

## 09h --- MOL00583--EGFR molecular-dynamics validation

09h follows the historical 09g choice and starts from the Vina-refined
MOL00583 pose exported by 09f. The receptor is extracellular EGFR chain
B from 3NJP.

The 1 ns production run remains a valid simulation of the configuration
that was actually prepared. Historical local metrics (including
approximately 2.09 Å mean ligand heavy-atom RMSD and 100% retention by
the notebook's local-region criterion) refer to the region surrounding
the propagated 09f pose; they must not be equated with the native EGF
interface.

### A26b --- native EGF-interface check

A26b reanalyzed the existing production trajectory without rerunning MD.
All 43 native-interface residues were mapped. After discarding the first
20%, 400 frames were analyzed:

``` text
EGF-interface contact fraction = 0.000000
minimum ligand–interface distance = 9.167 Å
mean minimum distance = 11.382 Å
median minimum distance = 11.386 Å
native-interface residues contacted = 0
```

Thus 09h supports persistence/reorganization in the **original off-site
EGFR region**, not retention at the native EGF--EGFR interface. This
does not exclude other possible MOL00583 binding modes; it means this
specific trajectory provides no evidence for native EGF-interface
binding.

------------------------------------------------------------------------

## 09i --- MOL00583--EGFR endpoint energetics

09i analyzes snapshots from the **1 ns 09h production trajectory** with
a single-trajectory GBn2/ACE endpoint protocol.

The 50-snapshot production result is approximately:

``` text
mean endpoint proxy = -20.10 kcal/mol
SD across snapshots = 2.37 kcal/mol
```

The SD describes variation across correlated snapshots, not uncertainty
of the mean. The calculation omits entropy, independent-component
relaxation, replicas and experimental calibration. It is not rigorous
binding ΔG and cannot be converted into Kd, Ki or evidence of
inhibition.

### Interpretation after A26b

Because A26b establishes that the analyzed 09h trajectory is outside the
native EGF interface, the 09i endpoint value characterizes the
**simulated off-site EGFR--MOL00583 configuration**. It must not be
presented as an energetic estimate of MOL00583 binding to the native EGF
site.

The saved calculation remains an internally consistent characterization
of the configuration actually simulated; the correction concerns its
biological/site interpretation.

------------------------------------------------------------------------

## 09j --- MOL00583 free in water, Windows/CUDA

09j characterizes the neutral MOL00583 microstate **free in water** with
Windows/CUDA replicate MD. The production configuration uses three
stochastic replicas up to 10 ns each, with different seeds but the same
initial molecular geometry.

The analysis includes ligand conformational RMSD, heavy-atom geometric
radius and hydration diagnostics. It performs no ΔG, entropy or affinity
calculation.

### Interpretation after A24--A26b

The free-water simulations remain valid as conformational/hydration
characterization. However, the historical `free` versus `bound`
comparison must be interpreted carefully: the 09h comparator is the
**EGFR-associated off-site configuration** identified by A26b, not a
validated EGF-interface-bound state.

``` text
09j free state = MOL00583 in water
historical "bound" reference = simulated off-site EGFR–MOL00583 configuration
free-versus-bound comparison ≠ thermodynamic EGF-site binding analysis
```

------------------------------------------------------------------------

## 09k --- MOL00583--EGFR bound-complex replicas on Windows/CUDA

09k extends the EGFR--MOL00583 configuration inherited from 09h on
Windows/CUDA.

Production:

``` text
3 replicas
10 ns production / replica
100 ps NPT before each production
distinct seeds / new velocities
shared 09h structural starting point
```

The replicas are stochastic branches from one starting configuration,
not independently prepared binding poses. Historical local-region
metrics describe motion relative to the residues neighboring the initial
09h/09f pose, which is now known to be off-site.

### A26c --- native EGF-interface relevance

A26c analyzed only the already existing 09k trajectories; **no MD,
docking or Vina calculation was rerun**. After discarding the first 20%,
800 frames per replica were analyzed, for 2400 frames total.

  -----------------------------------------------------------------------------
  Replica         Frames   EGF-interface Min distance     Mean min    Interface
                                 contact          (Å) distance (Å)     residues
                                fraction                              contacted
  --------- ------------ --------------- ------------ ------------ ------------
  1                  800        0.000000        9.040       13.174            0

  2                  800        0.000000        7.733       13.159            0

  3                  800        0.000000        9.796       13.579            0
  -----------------------------------------------------------------------------

No native EGF-interface contact was observed in **0/2400 analyzed
frames**. Even the closest approach (7.733 Å) remained above the 4.5 Å
A26c contact cutoff.

Therefore, the 09k simulations support persistence/evolution of the
previously identified **off-site EGFR--MOL00583 configuration**, rather
than retention at or spontaneous migration to the native EGF interface
during these simulations.

A26c outputs are preserved in the production run:

``` text
09k_A26c_EGF_interface_replica_summary.csv
09k_A26c_EGF_interface_framewise.csv
09k_A26c_EGF_interface_residue_occupancy.csv
```

The historical `TIER_1_PRIMARY_COMPUTATIONAL_LEAD` label is retained
only as provenance of the original 09g decision.

------------------------------------------------------------------------

## Pipeline 5 funnel

``` text
09a  MD-derived pharmacophore extraction
 ↓
09b  consolidated screening model
 ↓
09c  REINVENT4 generation/acquisition + downstream pharmacophore screening
 ↓
09d  chemistry filtering
 ↓
09e  whole-chain DiffDock ranking
 ↓
A24  native EGF-interface relevance check
 ↓
09f  local Vina refinement
 ↓
A25  receptor-frame displacement check
 ↓
09g  historical multi-criterion ranking
 ↓
A26  revised evidence interpretation
 ↓
09h  1 ns MD of historical MOL00583 configuration
 ↓
A26b 0/400 frames contact native EGF interface
 ↓
09i  endpoint characterization of that off-site trajectory
 ↓
09j  free MOL00583 replicas
 ↓
09k  3 × 10 ns EGFR–MOL00583 replicas
 ↓
A26c 0/2400 frames contact native EGF interface
```

The audit preserves the historical 09g ranking but separates it from
native-interface validation. MOL00583 remains the historical lead;
MOL00484 retains site-relevant structural support for follow-up. No
automatic reranking is imposed.

------------------------------------------------------------------------

## Software and environments

### Main PEARL analysis environment

Typical Python requirements include:

``` text
numpy
pandas
matplotlib
scipy
RDKit
MDAnalysis
BioPython
```

Stages 09h--09i use the dedicated environment below, in addition to the
upstream structural-bioinformatics tools.

### OpenMM / 09h--09i Jupyter environment

The verified kernel is **Python (PEARL MD 09h)**, Conda environment
`pearl-09h`. If it already exists, reuse it. To create it on a
compatible Conda platform, run these once in Terminal:

``` bash
conda create -n pearl-09h -c conda-forge python=3.11 openmm pdbfixer mdtraj openff-toolkit openff-forcefields openmmforcefields ambertools rdkit numpy pandas matplotlib jupyterlab ipykernel
conda activate pearl-09h
python -m ipykernel install --user --name pearl-09h --display-name "Python (PEARL MD 09h)"
jupyter lab
```

Select that kernel **inside Jupyter**; activating an environment in
Terminal alone does not switch an existing notebook. Package resolution
is platform-dependent; actual versions and input hashes are recorded per
run in `provenance.json`.

If AmberTools is installed but OpenFF cannot find `antechamber`, keep
this cell before 09h ligand parameterization:

``` python
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

This corrects executable discovery; it does not install missing
software. 09i reuses saved charges and does not invoke AmberTools.

Use `PLATFORM="CPU"` for the verified Mac setup. The OpenCL plugin was
present but device initialization returned
`No compatible OpenCL platform is available`; changing the platform
string alone does not enable a GPU. 09i defaults to `CPU_THREADS=4`. For
another machine, test GPU context creation before running a full
calculation.

### Windows / CUDA environment for 09j--09k

The portable Windows project root is:

``` text
C:\Users\Roberto\PEARL
```

The Windows follow-up notebooks are intended to use:

``` text
OS: Windows
GPU: NVIDIA RTX 4090
Jupyter kernel: Python (PEARL GPU)
```

The portable package includes selected 09f/09g inputs plus the
parametrized 09h system, portable XML states and the solute trajectory
needed for downstream analysis. It does not automatically install
software or reproduce every file from the Mac environment.

Before a Windows run:

1.  extract the portable package without creating a nested second
    `PEARL` directory;
2.  verify all transferred files against `MANIFEST_SHA256.json`;
3.  point notebook `PROJECT_ROOT` to `Path(r"C:\Users\Roberto\PEARL")`;
4.  verify NVIDIA driver, CUDA-enabled OpenMM context and the
    `Python (PEARL GPU)` kernel before running production;
5.  do not regenerate ligand chemistry unless explicitly required.

The portable 09h state files include:

``` text
production_system.xml
production_integrator.xml
equilibrated.xml
production_final.xml
physical_system.xml
```

The package preserves the 09h system and the neutral MOL00583
microstate. Binary Mac checkpoints were not transferred. XML portability
does not imply bitwise-identical stochastic continuation.

09j and 09k are designed so that the existing parametrized system can be
reused without requiring AmberTools/OpenFF/RDKit for the basic Windows
production benchmark, provided the transferred hashes and chemistry
audit pass.

### REINVENT4

REINVENT4 was run in a separate environment and its generated SMILES
were imported into 09c.

The production sampling configuration used the PubChem prior and
requested 5000 unique molecules, with 1000 molecules subsequently
analyzed in 09c.

### DiffDock

DiffDock was run in a dedicated environment.

The Apple-Silicon setup used during this project included:

``` text
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

The first run may download the ESM2 model and generate SO(2)/SO(3)
lookup tables.

### AutoDock Vina / Meeko

A separate Conda environment was used:

``` bash
conda create -n pearl-vina python=3.10 -y
conda activate pearl-vina
conda install -c conda-forge vina meeko rdkit pandas numpy scipy gemmi -y
```

For receptor preparation, PDB 3NJP chain-B residue 172 contains two
alternate conformations with equal occupancy:

``` text
B:172 altloc A = 0.50
B:172 altloc B = 0.50
```

The production protocol deterministically selected:

``` text
B:172=A
```

during Meeko receptor preparation.

------------------------------------------------------------------------

## Suggested execution order

The notebooks should be run sequentially:

``` text
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
 ↓
post-selection Windows/CUDA follow-up
 ├── 09j  free MOL00583 in water
 └── 09k  bound MOL00583–EGFR replicas
```

Several stages depend on files exported by the previous notebook, so the
`outputs/` directory should be preserved between runs. For 09j--09k,
also preserve the transferred 09h topology, atom ordering, parameter
cache and XML states because these define the chemistry and system
identity used for the Windows follow-up.

09c, 09e and 09f contain external-tool stages:

``` text
09c → REINVENT4 sampling
09e → DiffDock inference
09f → AutoDock Vina / Meeko
```

The corresponding notebook prepares the external input files and then
re-imports the generated results for analysis.

------------------------------------------------------------------------

## Output directories

The main output roots are:

``` text
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
├── pipeline_5_pharmacophore_09i_mol00583_endpoint/
│   └── <unique_run>/
├── pipeline_5_09j_free_ligand/
│   └── <unique_run>/
└── pipeline_5_09k_bound_replicates/
    └── <unique_run>/
```

Each notebook writes some combination of:

``` text
tables/
plots/
structures/
reports/
inputs/
```

The final integrated table is:

``` text
outputs/pipeline_5_pharmacophore_09g_integrated_lead_selection/
tables/09g_integrated_lead_priority.csv
```

The 09g lead-selection report is:

``` text
outputs/pipeline_5_pharmacophore_09g_integrated_lead_selection/
reports/09g_pipeline5_final_report.md
```

------------------------------------------------------------------------

## Scientific caveats

Pipeline 5 is a **computational prototype**.

1.  REINVENT4 production is unconstrained molecular sampling followed by
    pharmacophore screening, not pharmacophore-conditioned RL.
2.  The strict pharmacophore hits failed chemistry filters; the docking
    panel consists of chemically acceptable near-misses.
3.  09e is whole-chain/full-surface docking; its historical ranking is
    not an EGF-interface ranking.
4.  A24 shows that the selected MOL00583 pose is off-site; MOL00484
    retains native-interface relevance among the four historical 09f
    finalists.
5.  09f is local Vina rescoring/refinement, not an independent global
    docking experiment.
6.  A25 receptor-frame displacement indicates limited local
    rearrangement, not binding-site stability.
7.  The six 09g criteria are partially dependent computational criteria,
    not six independent biological evidence streams.
8.  MOL00583 remains the **historical 09g lead**, not a validated native
    EGF-site lead.
9.  A26b: 0/400 analyzed 09h frames contact the native EGF interface.
10. 09i (\~-20.10 kcal/mol) characterizes the off-site 09h
    configuration; it is not rigorous ΔG, Kd, Ki or proof of inhibition.
11. 09j remains valid for free-water conformational/hydration
    characterization, but its historical `bound` comparator is off-site.
12. A26c: 0/2400 analyzed 09k frames contact the native EGF interface.
13. Different seeds do not establish convergence; the 09k replicas share
    one structural starting point.
14. Historical outputs and provenance labels are retained; the audit
    corrects interpretation rather than rewriting past computations.
15. Experimental binding, functional and selectivity assays remain
    necessary for biological validation.

------------------------------------------------------------------------

## Final result

Pipeline 5 implements a traceable computational funnel from
peptide-derived structural information to small-molecule generation,
screening, docking, local refinement and post-selection simulation. The
audit adds a critical distinction between **historical computational
prioritization** and **native EGF-interface relevance**.

Historical 09g ordering, preserved as provenance:

``` text
1. MOL00583  — historical 09g computational lead
2. MOL00484  — historical follow-up finalist
3. MOL00600  — historical follow-up finalist
4. MOL00273  — historical follow-up finalist
```

This is **not** a post-audit EGF-site ranking. A24 shows that the
selected MOL00583 pose is off-site, whereas MOL00484 retains a selected
pose relevant to the native EGF interface. A26 therefore supports
site-focused reassessment rather than automatic reranking.

The downstream audit is consistent:

``` text
09h / A26b: 0/400 analyzed frames contact the native EGF interface
09k / A26c: 0/2400 analyzed frames contact the native EGF interface
```

Thus 09h--09k characterize an **off-site EGFR--MOL00583 configuration**.
The simulations and endpoint calculations remain valid records of the
configuration actually studied, but they do not validate MOL00583
binding at the native EGF--EGFR interface.

The final interpretation explicitly separates pharmacophore provenance,
chemistry eligibility, whole-chain docking support, local refinement,
historical prioritization, native-interface relevance, dynamic
persistence, endpoint energetics and future experimental validation.

------------------------------------------------------------------------

## Project

**PEARL --- Peptide Extraction and AI-guided Refinement for Ligand
design**

Target system:

``` text
EGFR extracellular domain
PDB: 3NJP, receptor chain B
Upstream peptide source: EGF chain D interacting with EGFR chain B
```

This repository contains a research/educational computational workflow.
It is not intended for clinical use.
