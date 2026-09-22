<p align="center">
  <img src="./PEARL_pipeline_2_3_4_overview.png"
       alt="PEARL integrated overview — Pipelines 2, 3 and 4"
       width="100%">
</p>

# PEARL Pipeline 4 — Protein Language Models and AI-Guided Peptide Design

Pipeline 4 adds a protein-language-model and structure-conditioned design branch to PEARL. It combines ESM-2 and ProteinMPNN with the structural, molecular-dynamics and endpoint-energy validation strategy established in Pipelines 2 and 3.

The objective is not to replace physical or experimental validation. Pipeline 4 asks whether AI-generated peptide sequences can remain plausible, structurally compatible with the EGFR interface and competitive with the established PEARL references.

```text
ESM-2 sequence analysis
        ↓
ProteinMPNN constrained inverse folding
        ↓
ESM-2 / ProteinMPNN prioritisation
        ↓
FoldX + Rosetta FlexPepDock
        ↓
explicit-solvent MD
        ↓
MM/GBSA-like endpoint comparison
        ↓
ESMFold structural validation
        ↓
developability screening
        ↓
evidence integration
        ↓
final multi-objective decision support
```

All results are computational. They do not constitute experimental evidence of binding, affinity, inhibition, solubility or biological activity.

---

## Relationship with Pipelines 2 and 3

Pipeline 2 produced the 11-residue EGF-derived reference peptide and selected CLEAR-derived counterfactual candidates:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Pipeline 3 subjected these candidates to explicit-solvent MD and comparative endpoint-energy analysis. Their complementary reference roles are retained throughout Pipeline 4:

```text
F0010  11-residue reference peptide extracted from native EGF
CF02   strongest Pipeline-3 dynamic-stability reference
CF06   strongest Pipeline-3 structural / endpoint-energy reference
```

Pipeline 4 adds an complementary AI design branch while preserving the central PEARL principle that no single score is treated as a universal measure of peptide quality.

---

## Biological and structural system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Interface:** chains `B–D`
- **Receptor chain:** `B`
- **Peptide chain:** `D`
- **11-residue EGF-derived reference peptide:** `F0010 = IGERCQYRDLK`
- **CLEAR references:** `CF06` and `CF02`

ProteinMPNN redesigns peptide chain `D` while receptor chain `B` remains fixed. Six peptide positions supported by previous FoldX and MD hotspot analyses are protected; the remaining positions are designable.

---

## Notebooks and execution order

1. `08a_ESM2_Peptide_Sequence_Scoring.ipynb`
2. `08b_ProteinMPNN_Structure_Conditioned_Peptide_Design.ipynb`
3. `08c_ESM2_ProteinMPNN_Integrated_Candidate_Selection.ipynb`
4. `08d_ProteinMPNN_Candidate_Structural_Validation.ipynb`
5. `08e_Top_ProteinMPNN_Lead_MD_Validation.ipynb`
6. `08f_ProteinMPNN_Endpoint_Energy_Comparison.ipynb`
7. `08g_ESMFold_Structural_Validation.ipynb`
8. `08h_Solubility_Developability_Screening.ipynb`
9. `08i_Bayesian_Optimisation_BoTorch.ipynb`
10. `08j_Final_Multi_Objective_Ranking.ipynb`

Notebooks `08a–08d` primarily use the PEARL AI environment. Notebooks `08e–08f` use the molecular-dynamics environment. Notebooks `08g–08j` integrate external structure predictions and the validated tables produced by the preceding stages.

**Canonical-version note.** The final repository should publish the audited production generations of 08g–08j. In particular, the canonical 08h/08i/08j correspond to the later full-cohort/evidence-tier generations; earlier proxy-only, readiness-only and two-candidate copies are retained only as version history and must not be used to describe the final Pipeline-4 state.

---

# Notebook summaries

## 08a — ESM-2 sequence plausibility

`08a` uses `esm2_t6_8M_UR50D` as a pretrained sequence-plausibility prior. It calculates masked pseudo-log-likelihoods, pseudo-perplexity, position-wise support, embeddings, cosine similarities and PCA coordinates.

The established-reference ranking was:

| Candidate | Mean masked PLL | Pseudo-perplexity |
|---|---:|---:|
| **CF02** | **−3.120** | **22.65** |
| CF06 | −3.232 | 25.32 |
| F0010 | −3.240 | 25.54 |

ESM-2 plausibility is not receptor-binding evidence.

## 08b — ProteinMPNN constrained design

`08b` performs structure-conditioned sequence generation on the EGFR–peptide complex. The production run generated:

```text
100 sampled sequences
24 unique peptide designs
```

The strongest recurring motif was centred on `TGPRNQYRDLX`, and the top ProteinMPNN sequence was `TGPRNQYRDLP`. ProteinMPNN explores sequence preferences conditioned on the chosen backbone and design constraints; it is not an affinity predictor.

## 08c — Integrated AI shortlist

`08c` combines ProteinMPNN score, sampling frequency, ESM-2 masked PLL and locality to F0010. The integrated score is a transparent prioritisation heuristic, not a physical energy.

| Candidate | Sequence | Integrated AI score |
|---|---|---:|
| **MPNN_NEW_01** | `TGPRNQYRDLP` | **0.601** |
| MPNN_NEW_02 | `IGPRNQYRDLG` | 0.551 |
| MPNN_NEW_04 | `IGPRNQYRDLN` | 0.514 |
| **MPNN_NEW_05** | `IGPRHQYRDLP` | 0.509 |
| MPNN_NEW_03 | `PGPRNQYRDLP` | 0.402 |

All five candidates were novel relative to the local CLEAR sequence space used in the preceding PEARL stages.

## 08d — FoldX and Rosetta validation

`08d` validates all five AI-derived candidates with FoldX and Rosetta FlexPepDock. Production Rosetta refinement uses `nstruct = 20` per candidate, for 100 decoys in total.

The integrated static structural ranking was:

```text
1. MPNN_NEW_05
2. MPNN_NEW_01
3. MPNN_NEW_02
4. MPNN_NEW_04
5. MPNN_NEW_03
```

Key results:

| Candidate | FoldX interaction energy | Rosetta best I_sc |
|---|---:|---:|
| **MPNN_NEW_05** | **≈ −13.75 kcal/mol** | **≈ −56.12** |
| MPNN_NEW_01 | ≈ −11.74 kcal/mol | ≈ −54.17 |

`MPNN_NEW_05` and `MPNN_NEW_01` were advanced to MD.

## 08e — Explicit-solvent MD

`08e` runs 1 ns explicit-solvent MD for the two advanced ProteinMPNN candidates and compares them with the existing Pipeline-3 results.

```text
AMBER ff14SB
TIP3P water
0.15 M NaCl
300 K, 1 bar
2 fs timestep
100 ps NVT → 100 ps NPT → 1 ns production
500 production frames
```

| Candidate | Mean peptide RMSD (Å) | Mean peptide RMSF (Å) | Persistent contacts ≥50% | Mean contact persistence |
|---|---:|---:|---:|---:|
| **CF02** | **0.901** | **0.737** | 43 | **0.603** |
| **MPNN_NEW_01** | 1.268 | 0.920 | 41 | 0.599 |
| CF06 | 1.293 | 0.904 | **45** | 0.529 |
| MPNN_NEW_05 | 1.417 | 0.831 | 40 | 0.535 |
| F0010 | 1.966 | 1.164 | 43 | 0.541 |

Among the two AI-derived candidates, the short-timescale dynamic comparison favoured `MPNN_NEW_01`.

## 08f — Endpoint-energy comparison

`08f` evaluates 50 uniformly distributed snapshots per new candidate using a single-trajectory MM/GBSA-like endpoint proxy:

```text
ΔE_endpoint = E_complex − E_receptor − E_peptide
```

| Rank | Candidate | Mean ΔE endpoint (kcal/mol) | SD (kcal/mol) |
|---:|---|---:|---:|
| **1** | **CF06** | **−65.66** | 6.80 |
| 2 | F0010 | −59.46 | 5.96 |
| 3 | CF02 | −56.62 | 4.63 |
| **4** | **MPNN_NEW_05** | **−55.57** | 5.61 |
| 5 | MPNN_NEW_01 | −53.43 | 4.41 |

`MPNN_NEW_05` is the endpoint-energy-favoured AI-derived candidate. The endpoint quantity is comparative and must not be interpreted as absolute binding free energy.

## 08g — ESMFold structural validation

`08g` validates externally generated ESMFold structures, confirms sequence identity and performs explicit comparison with the bound reference. It reports both confidence and geometric agreement rather than relying on pLDDT alone.

| Candidate | Mean pLDDT | Global Cα RMSD (Å) | Protected Cα RMSD after global fit (Å) | Mapped positions |
|---|---:|---:|---:|---:|
| **MPNN_NEW_01** | **72.56** | **4.49** | **3.67** | 11/11 |
| MPNN_NEW_05 | 67.27 | 5.07 | 3.83 | 11/11 |

Both sequence and structural-file QC passed. Within this comparison, `MPNN_NEW_01` is more confident and geometrically closer to the bound reference. For an isolated 11-residue peptide, pLDDT and RMSD remain structural descriptors—not affinity measurements.

## 08h — Solubility and developability screening

The canonical `08h` full-cohort run evaluates **27 sequences**: F0010, CF02, CF06 and the 24 unique ProteinMPNN designs. It computes sequence-derived physicochemical descriptors and imports an authentic **CamSol intrinsic** run with explicit provenance, sequence hashes and cohort validation.

CamSol is used here as a **computational solubility prediction**, not as an experimental solubility measurement. The imported run passed the notebook provenance and cohort-consistency checks:

```text
CamSol status = executed_imported_and_validated
```

The downstream developability contract therefore uses:

```text
developability_metric_used   = camsol_overall_score
developability_metric_source = CamSol computational prediction
```

Representative validated CamSol overall scores include F0010 = 2.0515, CF02 = 2.2462 and CF06 = 2.0956. The 24-design ProteinMPNN cohort is retained for downstream full-cohort screening and Bayesian decision support.

## 08i — Bayesian optimisation on the full ProteinMPNN cohort

The canonical full-cohort `08i` uses the **24 unique ProteinMPNN designs** as a discrete design pool. Five candidates already possess real structural evaluations from 08d and are used as the observed training set; the remaining **19 candidates are unobserved** at this expensive structural level.

The model inputs are three complementary cohort-wide computational predictors:

```text
ProteinMPNN favourability
ESM-2 PLL
CamSol overall score
```

They are not assumed to be statistically or experimentally independent. The supervised decision-support target is constructed from the five observed FoldX/Rosetta structural evaluations. It is **not binding affinity or free energy**.

A BoTorch `SingleTaskGP` is fitted to the five observed candidates, and `LogExpectedImprovement` is evaluated directly over every unobserved member of the discrete pool. Continuous sequence-space optimisation is intentionally avoided.

The saved run proposes:

```text
MPNN_POOL_023 = VGARNQYRDLN
status: proposed_for_next_08d_structural_validation
```

This sequence already belongs to the ProteinMPNN pool. It is **not a newly generated peptide, a validated lead or an affinity prediction**; it is the next candidate suggested for an expensive structural evaluation. The run records 24 pool members, 5 observations and 19 unobserved candidates, with all corresponding QC checks passing.

## 08j — Final multi-objective decision support with evidence tiers

The canonical `08j` uses an **evidence-tier** architecture so that candidates are compared only with information available at the corresponding level of evaluation.

**Tier 1 — Full 24-design screening.** All 24 ProteinMPNN designs are compared using cohort-wide ProteinMPNN, ESM-2 and validated CamSol computational signals.

**Tier 2 — Five structurally validated candidates.** The five candidates evaluated in 08d are compared using the available static structural evidence together with the AI and CamSol information.

**Tier 3 — Two fully evaluated new candidate peptides.** `MPNN_NEW_01` and `MPNN_NEW_05` are the two new candidates with the complete computational evidence set used at this stage: static structure, short MD, endpoint energy, ESMFold structural descriptors and CamSol prediction.

The Tier-3 criteria are complementary computational signals and are not assumed to constitute statistically or experimentally independent evidence. With only two candidates, standardized components encode directional contrasts rather than calibrated effect sizes. The saved Tier-3 decision-support score is:

```text
MPNN_NEW_01   tier3_score =  0.2
MPNN_NEW_05   tier3_score = -0.2
```

This score is a transparent multi-objective prioritisation aid, **not a validated therapeutic ranking, affinity estimate or biological superiority claim**. The endpoint-energy comparison itself favours `MPNN_NEW_05`, illustrating why the individual evidence domains must remain visible.

---

# Detailed methodology and data contracts

## Detailed 08a operations

For each sequence, `08a` masks every amino-acid position independently and records the probability assigned by ESM-2 to the original residue. The resulting position-wise log-probabilities are aggregated into a mean masked pseudo-log-likelihood and a pseudo-perplexity.

The notebook also performs:

- validation of input sequences and lengths;
- mutation-site comparison against F0010;
- residue-level support analysis;
- extraction of sequence embeddings;
- cosine-similarity analysis;
- PCA visualisation of the local sequence space;
- integration with the established PEARL candidate identifiers.

The embedding comparison is descriptive. The three reference sequences are short and highly similar, so small distances should not be interpreted as distinct biological states.

## Detailed 08b design protocol

The ProteinMPNN stage requires:

- a receptor–peptide PDB containing chains `B` and `D`;
- explicit chain assignments;
- a fixed receptor chain;
- a designable peptide chain;
- a position-specific fixed-residue dictionary;
- ProteinMPNN model weights;
- sampling temperatures and sample counts;
- parsing and deduplication of generated FASTA output.

The constrained design preserves the sequence positions supported by prior PEARL structural and dynamic evidence. This keeps the inverse-folding search connected to the experimentally motivated interface motif while allowing exploration at less protected sites.

Production generation used two sampling temperatures:

```text
0.1
0.2
```

The 100 generated samples were reduced to 24 unique sequences. Sampling frequency is retained as descriptive evidence but is not treated as a thermodynamic population.

## Detailed 08c prioritisation

For each unique ProteinMPNN design, `08c` preserves the original metrics and computes a transparent prioritisation score from:

```text
30% ProteinMPNN score
25% ESM-2 masked PLL
20% ProteinMPNN sampling frequency
25% sequence locality to F0010
```

The notebook also checks:

- candidate-ID uniqueness;
- exact sequence identity;
- novelty relative to the local CLEAR/04c sequence set;
- mutation counts relative to F0010;
- consistency between ProteinMPNN output and ESM-2 scoring input;
- diversity within the five-candidate shortlist.

The score is used to allocate expensive downstream computation. It has no physical unit and is not trained against experimental affinity.

## Detailed 08d structural workflow

### FoldX phase

For every shortlisted candidate, the notebook:

1. maps the designed peptide sequence onto the F0010-bound structural context;
2. creates candidate-specific mutation instructions;
3. prepares or builds the mutated receptor–peptide complex;
4. verifies the resulting chain and residue identities;
5. runs or imports FoldX `AnalyseComplex` results;
6. extracts receptor–peptide interaction energies;
7. preserves candidate-specific structures and provenance;
8. generates a FoldX ranking.

### Rosetta FlexPepDock phase

The FoldX-prepared structures are refined with high-resolution peptide docking. For each of the five candidates:

```text
nstruct = 20
```

The notebook parses Rosetta score files, associates every decoy with its candidate, identifies the best `I_sc`, checks the expected decoy count and creates the final static structural comparison.

FoldX and Rosetta use different energy functions. Their numerical values are therefore kept in separate columns; they are combined only through an explicitly labelled ranking procedure.

## Detailed 08e molecular-dynamics workflow

The two selected AI-derived candidates are prepared with the same general protocol used for the Pipeline-3 reference simulations. The workflow includes:

1. candidate-specific complex preparation;
2. solvation in TIP3P water;
3. addition of ions to approximately 0.15 M NaCl;
4. energy minimisation;
5. 100 ps NVT equilibration;
6. 100 ps NPT equilibration;
7. 1 ns production at 300 K and 1 bar;
8. trajectory and state-data export;
9. receptor-aligned peptide analysis.

The analysis calculates:

- peptide Cα RMSD after receptor alignment;
- per-residue and mean peptide Cα RMSF;
- receptor–peptide heavy-atom contacts;
- contact occupancy across the trajectory;
- contacts present in at least 50% of frames;
- mean contact persistence;
- temperature, density and potential-energy QC.

The reference trajectories are not rerun. Their official Pipeline-3 summaries are imported to preserve comparability and avoid silently mixing newly calculated and historical values.

## Detailed 08f endpoint workflow

Fifty snapshots are selected uniformly from each 1 ns production trajectory. For every snapshot, the endpoint calculation evaluates complex, receptor and peptide energies from the same geometry using the same GBn2 implicit-solvent endpoint protocol used for the Pipeline-3 reference candidates, enabling a harmonised cross-candidate comparison.

This single-trajectory construction reduces internal-coordinate noise but does not account for independent receptor or peptide relaxation. The notebook preserves:

- per-snapshot complex energy;
- per-snapshot receptor energy;
- per-snapshot peptide energy;
- per-snapshot endpoint difference;
- mean, median, standard deviation, minimum and maximum;
- number of successfully analysed snapshots;
- links to the cross-method evidence table.

The 50 snapshots originate from one short trajectory and are time-correlated. They must not be treated as 50 independent biological replicates.

## Detailed 08g structural-validation workflow

`08g` accepts externally generated candidate PDB files and performs a strict validation sequence before calculating structural metrics.

### Input and provenance checks

For each candidate, it records or verifies:

- `candidate_id`;
- expected sequence;
- source PDB path;
- copied pipeline PDB path;
- PDB SHA-256 hash;
- predicted chain;
- atom count;
- expected and observed residue counts;
- observed PDB sequence;
- exact sequence match.

### Confidence metrics

Where pLDDT is encoded in the PDB B-factor field, the notebook calculates:

- mean pLDDT;
- median pLDDT;
- minimum and maximum pLDDT;
- fraction of residues below 50;
- fraction of residues at or above 70;
- explicit pLDDT-validity status.

### Bound-reference comparison

The predicted peptide is mapped to chain `D` of the bound PEARL reference. The notebook records:

- reference PDB and chain;
- reference sequence;
- mapped position count;
- mapping coverage;
- global Cα RMSD;
- global backbone RMSD after Cα fitting;
- protected-position Cα RMSD after global fitting;
- protected-position Cα RMSD after local fitting;
- global and protected comparison status;
- aligned output PDB.

The distinction between global and local fitting is important. Global fitting describes overall conformational agreement, whereas protected-position local fitting asks whether the protected motif can adopt a similar local geometry independently of the overall peptide orientation.

The current production results were:

| Metric | MPNN_NEW_01 | MPNN_NEW_05 |
|---|---:|---:|
| Mean pLDDT | **72.56** | 67.27 |
| Fraction pLDDT ≥70 | **0.727** | 0.182 |
| Global Cα RMSD (Å) | **4.49** | 5.07 |
| Global backbone RMSD (Å) | **4.53** | 4.91 |
| Protected RMSD after global fit (Å) | **3.67** | 3.83 |
| Protected RMSD after local fit (Å) | **3.46** | 3.55 |
| Mapping coverage | 1.00 | 1.00 |

All comparisons completed successfully, but the magnitude of the RMSDs also warns that neither isolated prediction reproduces the bound conformation exactly.

## Detailed 08h screening and CamSol contract

### Canonical full-cohort contract

The canonical run contains 27 sequence records: F0010, CF02, CF06 and 24 unique ProteinMPNN designs. F0010 (`IGERCQYRDLK`) is an **11-residue reference peptide extracted from the native EGF sequence** (chain D, residues 38–48 in the adopted 3NJP mapping); it is not treated as an experimentally validated autonomous natural ligand. Legacy provenance labels are retained only where needed for historical traceability.

For every sequence, 08h validates candidate identity, canonical amino acids and SHA-256 sequence hashes and computes Biopython physicochemical descriptors.

### CamSol

The canonical run imports authentic CamSol results in **intrinsic** mode and validates the imported cohort against candidate IDs, sequences/hashes and provenance metadata. The final state is:

```text
camsol_status = executed_imported_and_validated
```

CamSol is a computational prediction and must not be described as measured solubility. The downstream metric is explicitly:

```text
developability_metric_used   = camsol_overall_score
developability_metric_source = CamSol computational prediction
```

The final QC reports candidate uniqueness, canonical sequences, hashes, finite proxy metrics, consistent CamSol provenance and explicit downstream metric source as passing.

## Detailed 08i full-cohort Bayesian optimisation

The canonical 08i run operates on the discrete 24-design ProteinMPNN cohort. Five candidates have observed FoldX/Rosetta structural evaluations and 19 remain unobserved at that level.

The cohort-wide features are ProteinMPNN favourability, ESM-2 PLL and validated CamSol overall score. Features are scaled over the complete pool. A BoTorch `SingleTaskGP` is fitted in double precision to the five observed candidates.

The acquisition function is `LogExpectedImprovement`. It is evaluated directly at every unobserved sequence rather than by continuous optimisation, because the candidate set is discrete.

The saved proposal is:

```text
MPNN_POOL_023
VGARNQYRDLN
```

for the next 08d structural evaluation. This is a decision-support proposal from the existing ProteinMPNN pool, not a newly generated sequence and not an affinity prediction.

The canonical QC confirms:

```text
full_pool_24                              True
validated_camsol_complete                 True
five_real_structural_observations         True
nineteen_unobserved_pool_candidates       True
botorch_gp_fitted                         True
proposal_is_existing_unobserved_sequence  True
ALL QC PASSED                             True
```

## Detailed 08j evidence-tier workflow

The canonical 08j run avoids comparing candidates as though they all had the same amount of evidence.

### Tier 1 — full 24-design screening

All 24 ProteinMPNN designs can be compared using the cohort-wide ProteinMPNN, ESM-2 and validated CamSol computational signals.

### Tier 2 — five structurally validated candidates

The five 08d candidates additionally possess real FoldX/Rosetta structural evaluations and can therefore be ranked at a richer evidence level.

### Tier 3 — two fully evaluated new candidate peptides

`MPNN_NEW_01` and `MPNN_NEW_05` additionally possess the complete computational set used here: static structural evaluation, 1 ns MD, harmonised endpoint-energy comparison, ESMFold descriptors and CamSol prediction.

These are complementary computational criteria rather than statistically or experimentally independent evidence classes. Equal weighting in the Tier-3 summary is a transparent decision-support convention, not a learned biological model. With two candidates, standardized values are directional contrasts rather than calibrated effect sizes.

All final claims remain computational and require experimental validation.

---

# Cross-method interpretation

| Method | Primary interpretation | Does not establish |
|---|---|---|
| ESM-2 | sequence plausibility | receptor affinity |
| ProteinMPNN | backbone-conditioned sequence compatibility | stronger binding |
| FoldX | approximate static interaction energetics | experimental affinity |
| Rosetta FlexPepDock | structural refinement and interface score | experimental activity |
| MD | short-timescale stability and contact persistence | full convergence |
| Endpoint energy | comparative energetic proxy | absolute ΔG, Kd, Ki or IC50 |
| ESMFold | confidence and structural plausibility | bound-state affinity |
| GRAVY / physicochemical descriptors | sequence-derived developability triage | measured solubility |
| CamSol | intrinsic computational solubility prediction | experimental solubility |
| BoTorch | pilot GP/acquisition-based selection of the next costly evaluation | affinity prediction or an iterative closed-loop optimisation campaign |

No single method is sufficient on its own.

---

## Main AI-generated candidates

```text
MPNN_NEW_01 = TGPRNQYRDLP
MPNN_NEW_05 = IGPRHQYRDLP
```

### MPNN_NEW_01

Favoured by:

- 1 ns MD among the two new designs;
- ESMFold pLDDT;
- global and protected-reference RMSD;
- GRAVY developability proxy;
- equal-weight 08i/08j integration;
- three of four weight scenarios, with one additional tie.

### MPNN_NEW_05

Favoured by:

- FoldX static interaction energy;
- Rosetta FlexPepDock interface score;
- MM/GBSA-like endpoint energy among the two new designs.

Both remain Pareto-optimal computational hypotheses.

---

## Software and environments

### AI environment

Typical dependencies:

- Python 3.11;
- PyTorch;
- fair-esm;
- ProteinMPNN and model weights;
- NumPy, pandas and Matplotlib;
- scikit-learn;
- Biopython;
- FoldX and Rosetta FlexPepDock installations where required.

Primarily used for `08a–08d` and the sequence-derived parts of `08g–08j`.

### Molecular-dynamics environment

Typical dependencies:

- Python 3.11;
- OpenMM;
- PDBFixer;
- MDAnalysis;
- NumPy, pandas and Matplotlib.

Primarily used for `08e–08f`.

### Optional software

- **CamSol:** authentic intrinsic-mode results were imported and validated in the canonical 08h full-cohort run. These are computational predictions, not measured solubility.
- **BoTorch:** the canonical 08i full-cohort run fitted a pilot `SingleTaskGP` to five structurally observed candidates and evaluated `LogExpectedImprovement` over 19 unobserved ProteinMPNN pool members. This is a next-evaluation selector, not a complete iterative optimisation campaign.

---

## Output directories

Generated data are stored below `outputs/` in notebook-specific directories:

```text
pipeline_4_ai_08a_esm2_sequence_scoring/
pipeline_4_ai_08b_proteinmpnn_design/
pipeline_4_ai_08c_integrated_candidate_selection/
pipeline_4_ai_08d_structural_validation/
pipeline_4_ai_08e_top_mpnn_md/
pipeline_4_ai_08f_endpoint_energy/
pipeline_4_ai_08g_esmfold_structural_validation/
pipeline_4_ai_08h_solubility_developability/
pipeline_4_ai_08i_bayesian_optimisation/
pipeline_4_ai_08j_final_multiobjective/
```

Important downstream tables include:

```text
08g_ESMFold_structural_validation_summary.csv
08h_solubility_developability_summary.csv
08i_observed_integrated_evidence.csv
08i_weight_sensitivity.csv
08i_botorch_readiness.csv
08j_final_multiobjective_ranking.csv
08j_weight_scenario_outcomes.csv
08j_candidate_robustness.csv
08j_evidence_status.csv
```

The 08g–08j notebooks also produce QC tables, provenance JSON files, figures and Markdown reports. Large trajectories, Rosetta decoys, model weights and external software installations should not normally be committed unless intentionally archived.

## Detailed output inventory

### 08a

- sequence validation table;
- masked position-wise probabilities;
- sequence-level PLL and pseudo-perplexity;
- embeddings and similarity matrices;
- PCA coordinates and plots.

### 08b

- parsed receptor–peptide input structure;
- chain-assignment configuration;
- fixed-position dictionaries;
- ProteinMPNN FASTA output;
- sample-level and unique-sequence tables;
- amino-acid preference summaries.

### 08c

- integrated ESM-2/ProteinMPNN evidence table;
- novelty and locality tables;
- selected five-candidate manifest;
- ranking and shortlist figures.

### 08d

- candidate mutation inputs;
- FoldX-built structures;
- FoldX `AnalyseComplex` outputs;
- Rosetta command files and score files;
- refined PDB decoys;
- best-decoy and integrated structural rankings.

### 08e

- prepared and solvated systems;
- equilibration and production state data;
- DCD trajectories;
- RMSD, RMSF and contact-persistence tables;
- thermodynamic QC;
- imported reference comparison and final MD ranking.

### 08f

- selected snapshot manifest;
- per-snapshot energy decomposition;
- new-candidate endpoint summary;
- five-candidate endpoint ranking;
- cross-method evidence table;
- `08f_candidate_manifest.csv` for downstream sequence identity.

### 08g

- validated predicted PDB files;
- aligned PDB files;
- structural-validation summary;
- pLDDT and RMSD figures;
- PDB hashes and provenance;
- `08g_QC.csv` and report.

### 08h

- physicochemical descriptor summary;
- CamSol submission FASTA;
- CamSol manifest and CSV templates;
- optional validated CamSol residue profiles;
- developability figure;
- `08h_QC.csv`, provenance JSON and report.

### 08i

- full 24-design feature pool;
- five-candidate observed structural training set;
- 19-member unobserved discrete candidate pool;
- fitted BoTorch `SingleTaskGP`;
- `LogExpectedImprovement` scores over the unobserved pool;
- next-evaluation proposal (`MPNN_POOL_023`);
- QC, provenance and run report.

### 08j

- Tier-1 full 24-design screening;
- Tier-2 five-candidate structurally validated ranking;
- Tier-3 two-candidate complete computational comparison;
- evidence-tier status and provenance;
- multi-objective decision-support outputs;
- QC and final report.

---

# Interpretation and limitations

1. ESM-2 and ProteinMPNN are not affinity predictors.
2. FoldX and Rosetta scores are model-dependent approximations.
3. The 1 ns MD trajectories support short-timescale comparative screening but do not establish full conformational convergence.
4. The endpoint calculation is a harmonised single-trajectory GBn2/MM-GBSA-like proxy, not absolute binding free energy.
5. ESMFold confidence and RMSD do not demonstrate receptor binding.
6. GRAVY, charge, pI and related descriptors are sequence-derived developability proxies, not measured solubility.
7. CamSol was executed/imported and validated for the canonical full cohort, but remains a computational solubility prediction rather than an experimental measurement.
8. The BoTorch run is a small-sample pilot with five structural observations and 19 unobserved pool members. It selects a candidate for the next expensive evaluation; it does not establish affinity and is not a completed iterative optimisation campaign.
9. The computational evidence domains are complementary but are not assumed to be statistically or experimentally independent.
10. Multi-objective weights are transparent decision-support choices rather than experimentally learned biological weights.
11. Evidence tiers must be preserved: a candidate lacking an expensive downstream evaluation must not be assigned favourable values for missing evidence.
12. Experimental validation remains required for binding, affinity, inhibition, selectivity, solubility, stability, toxicity and cellular activity.

---

# Final Pipeline 4 conclusion

Pipeline 4 adds a complementary ESM-2 and ProteinMPNN design branch to PEARL and carries selected candidates through structural, dynamic, energetic, conformational, developability and decision-support analyses.

The established references remain:

```text
F0010  11-residue reference peptide extracted from native EGF
CF02   strongest Pipeline-3 dynamic-stability reference
CF06   strongest Pipeline-3 structural / endpoint-energy reference
```

The principal AI-derived candidates are:

```text
MPNN_NEW_01 = TGPRNQYRDLP
MPNN_NEW_05 = IGPRHQYRDLP
```

Their evidence is complementary rather than uniformly concordant. `MPNN_NEW_01` is favoured by several short-timescale dynamic and ESMFold structural descriptors, whereas `MPNN_NEW_05` is favoured by the harmonised endpoint-energy comparison and parts of the static structural evaluation.

The canonical full-cohort extension is now complete at the computational decision-support level:

```text
08h: 27-sequence cohort; authentic intrinsic CamSol imported and validated
08i: 24 ProteinMPNN designs; 5 observed structural candidates; 19 unobserved
     SingleTaskGP + LogExpectedImprovement executed
     next proposed structural evaluation: MPNN_POOL_023 = VGARNQYRDLN
08j: evidence-tier integration
     Tier 1 = 24 designs
     Tier 2 = 5 structurally evaluated candidates
     Tier 3 = 2 fully evaluated new candidate peptides
```

The BoTorch proposal is not a new sequence or validated lead; it identifies which existing pool member should receive the next costly structural evaluation. Likewise, CamSol is a computational prediction rather than measured solubility, and the Tier-3 score is a decision-support summary rather than an affinity or therapeutic ranking.

The methodological conclusion of Pipeline 4 is therefore not that one computational method identifies a universally superior peptide. Instead, PEARL preserves provenance, exposes disagreement among complementary computational criteria and allocates increasingly expensive evaluations through explicit evidence tiers.

All candidates remain computational hypotheses. Experimental work and, where appropriate, longer replicated simulations and more rigorous free-energy calculations are required before quantitative biological or therapeutic claims can be made.
