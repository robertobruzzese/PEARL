![PEARL — Peptide Extraction and AI-guided Refinement for Ligand design](pearl.png)
# PEARL — Peptide Extraction and AI-guided Refinement for Ligand Design

# PEARL — Peptide Extraction and AI-guided Refinement for Ligand Design

**From Dimers to Drugs**

PEARL is a computational drug-discovery research prototype that reverse-engineers a kinase protein–protein interface and uses structural bioinformatics, molecular modelling, molecular dynamics (MD), protein AI and cheminformatics to prioritize peptide and small-molecule inhibitor hypotheses.

The current implementation is developed around the **EGFR kinase-domain asymmetric dimer (PDB `3NJP`)**, focusing on the interface between chains **B and D**.

> **Repository status:** five implemented computational pipelines spanning interface analysis, peptide design, molecular dynamics, AI-guided peptide redesign, MD-derived pharmacophore construction and small-molecule lead prioritization. All reported leads remain computational hypotheses; no experimental binding, inhibition, efficacy or safety validation is claimed.

---

## Workflow at a glance

```text
EGFR kinase dimer (3NJP, chains B–D)
        ↓
Pipeline 1 — interface mapping and hotspot discovery
        ↓
Pipeline 2 — peptide miniaturization, CLEAR/VAE optimization,
             FoldX and Rosetta FlexPepDock validation
        ↓
Pipeline 3 — explicit-solvent MD, contact persistence and
             comparative endpoint energetics
        ↓
Pipeline 4 — ESM-2 and ProteinMPNN peptide design,
             structural/MD/endpoint validation, ESMFold,
             developability proxies and multi-objective integration
        ↓
Pipeline 5 — MD-derived pharmacophore, REINVENT4 sampling,
             pharmacophore screening, chemistry rescue,
             DiffDock, Vina local refinement and lead selection
        ↓
Prioritized peptide and small-molecule hypotheses
```

PEARL follows a layered-validation philosophy: no surrogate score, force-field energy, docking output, structure prediction or short MD trajectory is treated as sufficient evidence on its own. Cross-method agreement is used for **prioritization**, not as proof of biological activity.

---

## Biological system

| Item | Current implementation |
|---|---|
| Target | Epidermal Growth Factor Receptor (EGFR) |
| Reference structure | PDB `3NJP` |
| Studied interface | Chains `B–D` |
| Receptor chain | `B` |
| Partner/peptide chain | `D` |
| Short natural reference | `F0010` |
| F0010 sequence | `IGERCQYRDLK` |
| F0010 mapping | Chain D residues 38–48 |

F0010 is a contiguous, hotspot-rich fragment derived from the native interface and serves as the principal short-peptide reference throughout the repository.

---

## Repository structure

```text
PEARL/
├── README.md
├── pearl.png
├── pipeline_1_initial_prototype/
│   ├── README.md
│   └── 01–04b  interface graph, biological-interface validation,
│              FoldX alanine scanning, peptide extraction and ranking
├── pipeline_2_structural_validation_and_docking/
│   ├── README.md
│   ├── 02c–05e  peptide miniaturization, CLEAR, FoldX,
│   │             FlexPepDock and adjacent-pair scoring
│   └── vae_extension/
│       ├── README.md
│       └── 06a–06d  VAE training, latent analysis,
│                    latent counterfactuals and method comparison
├── pipeline_3_molecular_dynamics_and_binding_energy/
│   ├── README.md
│   ├── 07a–07e  OpenMM setup, interface/contact analysis,
│   │             peptide MD and endpoint-energy comparison
│   └── 10ns_continuation/
│       └── 07d_OpenMM_continue_from_1ns_to_10ns.ipynb
├── pipeline_4_protein_language_model_design/
│   ├── README.md
│   └── 08a–08j  ESM-2, ProteinMPNN, FoldX/Rosetta, MD,
│                 endpoint energetics, ESMFold, developability,
│                 BO readiness and multi-objective ranking
└── pipeline_5_pharmacophore_and_small_molecule_leads/
    ├── README.md
    └── 09a–09g  MD pharmacophore, consolidation, REINVENT4,
                  chemistry filtering, DiffDock, Vina and lead selection
```

Some computationally expensive notebooks have test and production variants. The pipeline-level README files document their execution order, inputs, parameters and output contracts.

---

## Implementation status

| Pipeline | Status | Implemented scope |
|---|---|---|
| **1 — Initial prototype** | Complete (historical branch) | Interface identification, graph construction, biological-interface checks, FoldX alanine scanning, hotspots and initial peptide windows |
| **2 — Peptide design and structural validation** | Complete | Hotspot-centered miniaturization, F0010 selection, 539-variant local landscape, GNN oracle, Direct CLEAR, FoldX, FlexPepDock, adjacent-pair scoring and VAE-CLEAR comparison |
| **3 — MD and endpoint energetics** | Complete as a comparative prototype | OpenMM explicit-solvent simulations, RMSD/RMSF, contact persistence, hotspot integration and single-trajectory MM/GBSA-like endpoint comparisons; a 10 ns continuation notebook is also included |
| **4 — AI-guided peptide design** | Implemented through `08j`, with explicit partial methods | ESM-2, ProteinMPNN, FoldX/Rosetta, 1 ns MD, endpoint energetics and ESMFold are implemented; `08h` uses developability proxies because authentic CamSol was not run; `08i` provides evidence integration and BO readiness, not an executed BoTorch optimization; `08j` provides descriptive multi-objective/Pareto decision support |
| **5 — Pharmacophore and small molecules** | Complete as a computational discovery funnel | MD-derived pharmacophore, screening-ready consolidation, unconstrained REINVENT4 generation followed by pharmacophore screening, chemistry rescue, DiffDock, Vina local refinement and integrated lead prioritization |

---

## Pipeline 1 — Interface and hotspot prototype

Pipeline 1 established the original PEARL logic:

```text
3NJP → B–D interface → residue-contact graph → FoldX alanine scan
     → energetic hotspots → contiguous peptide windows
     → initial candidate ranking
```

This branch is retained as the historical prototype and provenance for later structural and energetic choices.

---

## Pipeline 2 — Peptide design, CLEAR and structural validation

Pipeline 2 contains two complementary peptide-design branches.

### Hotspot-centered miniaturization

Natural interface fragments are ranked structurally and energetically, prepared for guided docking and refined with Rosetta FlexPepDock. The 11-residue peptide **F0010 (`IGERCQYRDLK`)** emerged as the primary miniaturized reference.

### Local CLEAR and VAE optimization

A constrained **539-variant** sequence landscape around F0010 supports a differentiable GNN-based local oracle and Direct CLEAR-inspired counterfactual search. FoldX and FlexPepDock are then used as explicit structural filters. A true VAE extension explores the same local landscape in latent space.

Direct CLEAR found 36 successful sequences and VAE-CLEAR found four; all four VAE successes were also recovered by Direct CLEAR. This is useful algorithmic convergence, but not independent biological validation because the methods share the underlying local dataset and oracle.

Important recurring mutation patterns include `D9E` and `C5S + D9E`. Rosetta results indicate that the effect of `D9E` depends strongly on the accompanying mutation.

---

## Pipeline 3 — Molecular dynamics and comparative endpoint energetics

Pipeline 3 moves from static models to explicit-solvent MD using OpenMM with AMBER ff14SB, TIP3P water, approximately 0.15 M NaCl, 300 K and 1 bar. The primary production comparison uses 1 ns trajectories; these are short comparative simulations, not evidence of long-timescale convergence.

### Interface-level results

- 147 unique B–D contacts were observed during the prototype MD.
- 80 contacts persisted in at least 50% of analyzed frames.
- All 11 F0010 positions were supported by MD-persistent interface contacts.
- Six of the 11 positions were supported by both FoldX hotspot analysis and MD persistence.

### Peptide comparison

| Candidate | Sequence | Mean RMSD (Å) | Mean RMSF (Å) | Persistent contacts ≥50% | Mean persistence | Endpoint proxy (kcal/mol) |
|---|---|---:|---:|---:|---:|---:|
| `CF02` | `IGERSQYRELK` | **0.901** | **0.737** | 43 | **0.603** | −56.62 ± 4.63 |
| `CF06` | `IGERCQYRELR` | 1.293 | 0.904 | **45** | 0.529 | **−65.66 ± 6.80** |
| `F0010` | `IGERCQYRDLK` | 1.966 | 1.164 | 43 | 0.541 | −59.46 ± 5.96 |

`CF02` is the strongest short-timescale dynamic-stability reference, whereas `CF06` is the strongest structural/endpoint-energy reference. The endpoint calculation is a comparative, single-trajectory **MM/GBSA-like proxy**; it is not a rigorous absolute binding free energy.

---

## Pipeline 4 — Protein AI and peptide redesign (`08a–08j`)

Pipeline 4 adds sequence- and structure-conditioned AI while preserving downstream physical and structural checks.

| Notebook | Implemented role | Scientific status |
|---|---|---|
| `08a` | ESM-2 masked pseudo-log-likelihood, pseudo-perplexity and embeddings | Sequence-plausibility prior; not binding evidence |
| `08b` | Constrained ProteinMPNN redesign of peptide chain D | 100 samples, 24 unique designs; not affinity prediction |
| `08c` | Integrated ESM-2/ProteinMPNN shortlist | Transparent heuristic prioritization |
| `08d` | FoldX and Rosetta FlexPepDock validation | Static structural/energetic filtering |
| `08e` | Explicit-solvent MD for top AI-derived peptides | 1 ns comparative prototype |
| `08f` | Endpoint-energy comparison | Comparative MM/GBSA-like proxy, not absolute ΔG |
| `08g` | ESMFold sequence/file QC, pLDDT and bound-reference geometry comparison | **Completed**; isolated-peptide confidence and geometry are not affinity measurements |
| `08h` | Physicochemical and developability screening | **Completed in proxy mode**; authentic CamSol was not executed |
| `08i` | Cross-notebook evidence contract, normalization, sensitivity analysis and BO readiness | **No true BoTorch optimization executed**: only two observations, no candidate pool, GP, acquisition function or iterative proposal loop |
| `08j` | Equal-domain ranking, sensitivity scenarios and Pareto analysis | **Completed descriptive multi-objective decision support**, not Bayesian optimization |

### AI-derived peptide results

The five-candidate ProteinMPNN shortlist was led initially by `MPNN_NEW_01` (`TGPRNQYRDLP`). Static FoldX/Rosetta analysis favored `MPNN_NEW_05` (`IGPRHQYRDLP`), and both advanced to MD.

| Evidence domain | `MPNN_NEW_01` | `MPNN_NEW_05` | Favored candidate |
|---|---:|---:|---|
| Mean peptide RMSD, 1 ns (Å) | **1.268** | 1.417 | `MPNN_NEW_01` |
| Mean contact persistence | **0.599** | 0.535 | `MPNN_NEW_01` |
| Endpoint proxy (kcal/mol) | −46.41 ± 4.66 | **−53.67 ± 6.43** | `MPNN_NEW_05` |
| ESMFold mean pLDDT | **72.56** | 67.27 | `MPNN_NEW_01` |
| ESMFold global Cα RMSD (Å) | **4.49** | 5.07 | `MPNN_NEW_01` |
| GRAVY proxy | **−1.94** | −1.44 | `MPNN_NEW_01` (more hydrophilic) |

The final equal-weight exploratory ranking favors `MPNN_NEW_01`; `MPNN_NEW_05` remains the endpoint-energy-favored alternative. Both are Pareto-optimal in the implemented two-candidate analysis.

This conclusion must remain conservative:

- CamSol is supported as an import contract, but no authentic CamSol result is reported; `08h` used Biopython-derived descriptors and GRAVY as a proxy.
- BoTorch readiness checks are present, but Bayesian optimization was disabled and unavailable in the executed state. No GP posterior, acquisition function, newly proposed sequence or iterative BO cycle was produced.
- With only two candidates, standardized utilities largely encode ordering rather than robust effect magnitude.

---

## Pipeline 5 — MD-derived pharmacophore and small-molecule leads (`09a–09g`)

Pipeline 5 translates persistent peptide–EGFR interactions into a screening hypothesis and a small-molecule prioritization funnel.

```text
300 peptide–EGFR MD frames
→ 5,330 active feature observations
→ 59 persistent candidate-level features
→ 32 spatial clusters
→ 17 core consensus features
→ 10 consolidated groups (5 mandatory, 4 optional, 1 contextual)
→ 1,000 REINVENT4 molecules analyzed; 996 embedded in 3D
→ 83 complete mandatory assignments; 2 strict hits
→ 0 strict hits chemically eligible
→ 8 chemically acceptable near-miss rescue candidates
→ 80 DiffDock poses
→ 4 Vina-refined finalists
→ integrated lead selection
```

### Generation and pharmacophore screening

REINVENT4 was used for **unconstrained de novo molecular sampling** (5,000 SMILES requested). A 1,000-molecule subset was then embedded and screened against the pharmacophore. The implemented workflow is therefore:

```text
REINVENT4 generation → downstream pharmacophore screening
```

It is **not pharmacophore-guided or pharmacophore-conditioned reinforcement learning**.

The two strict pharmacophore hits, `MOL00336` and `MOL00570`, failed downstream medicinal-chemistry filters. Consequently, the eight compounds advanced to docking are explicitly retained as:

```text
docking_track        = NEAR_MISS_RESCUE
pharmacophore_status = NEAR_MISS_MANDATORY_ONLY
chemistry_class      = CHEM_PASS
```

Their near-miss status is not erased by subsequent docking results.

### DiffDock and Vina

Eight near-miss rescue candidates were docked to EGFR chain B with DiffDock (10 poses per molecule). The top four were then evaluated by scoring the selected DiffDock pose with AutoDock Vina, applying short local Vina optimization and measuring local pose displacement. This is local refinement of DiffDock poses, not a new global Vina docking campaign.

| Candidate | DiffDock role | Vina after local refinement | Local RMSD (Å) | Integrated interpretation |
|---|---|---:|---:|---|
| `MOL00583` | DiffDock rank 2; broad receptor engagement | −4.314 | 0.586 | **Primary computational lead; broadest cross-method support** |
| `MOL00484` | Strongest engagement/chemistry profile | −4.267 | **0.057** | Orthogonal follow-up lead; exceptional local geometry stability |
| `MOL00600` | Strong pharmacophore fit | **−5.791** | 0.190 | Orthogonal follow-up lead; strongest Vina support |
| `MOL00273` | **Strongest original DiffDock ranking** | −3.937 | 0.203 | Method-specific/orthogonal follow-up lead |

Vina outputs are empirical docking scores for within-protocol comparison. They are **not MM-GBSA, MM-PBSA or rigorous binding free energies (ΔG)**.

### Final Pipeline 5 priorities

| Rank | Candidate | Priority | Top-two support axes | Status |
|---:|---|---|---:|---|
| 1 | `MOL00583` | Tier 1 | 4/6 | Primary computational lead |
| 2 | `MOL00484` | Tier 2 | 3/6 | Orthogonal follow-up |
| 3 | `MOL00600` | Tier 2 | 3/6 | Orthogonal follow-up |
| 4 | `MOL00273` | Tier 2 | 2/6 | Method-specific follow-up |

All four are Pareto non-dominated under the implemented rank-space analysis. None is a validated binder or drug candidate, and all retain near-miss rescue pharmacophore provenance.

---

## Current prioritized leads

### Peptides

| Candidate | Sequence | Current role |
|---|---|---|
| `CF06` | `IGERCQYRELR` | Strongest integrated structural and endpoint-energy reference |
| `CF02` | `IGERSQYRELK` | Strongest 1 ns dynamic-stability reference; CLEAR/VAE convergence |
| `MPNN_NEW_01` | `TGPRNQYRDLP` | Best equal-weight Pipeline 4 exploratory balance; stronger MD/developability-proxy/ESMFold profile among the two AI finalists |
| `MPNN_NEW_05` | `IGPRHQYRDLP` | Static FoldX/Rosetta and endpoint-energy-favored AI alternative |
| `F0010` | `IGERCQYRDLK` | Native interface-derived reference/control |

These candidates form a **multi-method shortlist**, not a universal affinity ranking.

### Small molecules

| Candidate | Current role |
|---|---|
| `MOL00583` | Primary computational lead through broad cross-method convergence |
| `MOL00484` | Chemistry, engagement and local-geometry follow-up |
| `MOL00600` | Pharmacophore-fit and Vina-score follow-up |
| `MOL00273` | DiffDock-favored orthogonal follow-up |

---

## Software and external dependencies

The repository uses multiple environments because several external tools have distinct installation and licensing requirements.

### Core scientific Python stack

- Python and Jupyter Notebook
- NumPy, pandas, SciPy and Matplotlib
- scikit-learn
- PyTorch
- Biopython
- NetworkX
- RDKit

### Structural modelling and peptide design

- FoldX
- Rosetta, including FlexPepDock
- ESM-2 / `fair-esm`
- ProteinMPNN
- ESMFold-generated structure inputs

### Molecular dynamics and trajectory analysis

- OpenMM
- PDBFixer
- MDAnalysis
- AMBER-family force fields and implicit-solvent models as specified by the notebooks

### Small-molecule generation and docking

- REINVENT4
- DiffDock and its PyTorch Geometric/e3nn dependencies
- AutoDock Vina
- Meeko

FoldX, Rosetta, ProteinMPNN, REINVENT4, DiffDock, Vina and model weights/databases may require separate installation, licenses, downloads and local path configuration. Exact versions, production/test switches and platform-specific setup should be recorded alongside each executed run for reproducibility.

CamSol and BoTorch should not be listed as completed computational dependencies of the current results: CamSol was not executed, and the BoTorch optimization path was not run.

---

## Interpretation and limitations

- **No experimental validation:** the repository contains no direct measurements of binding, inhibition, selectivity, cellular activity, toxicity, pharmacokinetics or efficacy.
- **Model-system scope:** conclusions are specific to the selected EGFR `3NJP` B–D structural context and the preparation choices used here.
- **Local peptide search:** CLEAR and VAE-CLEAR operate on a deliberately local landscape around F0010; the oracle learns computational labels rather than experimental affinity.
- **Shared evidence is not independence:** Direct CLEAR and VAE-CLEAR share data and an oracle; apparent convergence must not be overstated.
- **Approximate static scores:** FoldX energies and Rosetta scores are model-dependent and are not experimental affinities.
- **Short MD:** the principal 1 ns peptide trajectories support comparative prototyping but cannot establish long-timescale stability or convergence. Frames from one trajectory are time-correlated, not independent replicates.
- **Endpoint energetics:** the reported peptide endpoint values are comparative, single-trajectory MM/GBSA-like estimates, not rigorous absolute binding free energies.
- **ESMFold:** confidence and RMSD for isolated 11-residue peptides do not establish receptor-bound conformations or affinity.
- **Developability:** `08h` reports sequence-derived physicochemical descriptors and heuristics. GRAVY is a proxy, not CamSol and not an experimental solubility measurement.
- **Bayesian optimization:** `08i` does not contain an executed BoTorch GP/acquisition/iteration workflow. `08j` is transparent multi-objective decision support, not BO.
- **Pharmacophore generation:** REINVENT4 sampling was unconstrained; pharmacophore matching occurred downstream. It must not be described as pharmacophore-guided RL.
- **Near-miss rescue:** all docked and finalized small molecules are chemically acceptable mandatory-only near misses, not strict pharmacophore hits.
- **Docking:** DiffDock confidence, pose-support metrics and Vina scores are method-specific ranking signals. Vina is neither MM-GBSA nor rigorous ΔG.
- **Drug-likeness is not a drug:** cheminformatic filters and Pareto rankings identify follow-up hypotheses, not safe or effective medicines.

---

## Reproducibility principles

For every production result, preserve:

1. exact candidate identifiers and sequences/SMILES;
2. input structures and chain/residue mappings;
3. software and model versions;
4. random seeds and production/test switches;
5. executable/database/model-weight paths;
6. raw outputs before aggregation;
7. hashes or manifests where notebooks provide them;
8. explicit distinction between calculated values, imported external outputs and proxies.

The pipeline README files and notebook QC cells are the authoritative sources for stage-specific execution details.

---

## Final repository status

PEARL now implements a five-pipeline, end-to-end computational prototype:

1. the EGFR B–D interface is identified and characterized;
2. hotspot-rich peptides are extracted, miniaturized and locally optimized;
3. prioritized peptides are compared by static modelling, explicit-solvent MD and endpoint proxies;
4. ESM-2 and ProteinMPNN expand the peptide search, with FoldX/Rosetta, MD, endpoint and ESMFold follow-up plus conservative developability and multi-objective integration;
5. peptide–EGFR MD ensembles are translated into a pharmacophore and a small-molecule discovery funnel ending in one primary and three orthogonal computational leads.

The central result is not a validated inhibitor. It is a reproducible chain of computational evidence that narrows a kinase-interface design problem to a compact set of peptide and small-molecule hypotheses suitable for more rigorous simulation and, ultimately, experimental testing.peptides and molecular candidates are research hypotheses and are **not validated drugs, inhibitors or therapeutic compounds**.
