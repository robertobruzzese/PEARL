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

Pipeline 2 produced the natural short reference and selected CLEAR-derived counterfactual candidates:

```text
F0010 = IGERCQYRDLK
CF06  = IGERCQYRELR
CF02  = IGERSQYRELK
```

Pipeline 3 subjected these candidates to explicit-solvent MD and comparative endpoint-energy analysis. Their complementary reference roles are retained throughout Pipeline 4:

```text
F0010  natural / miniaturised reference
CF02   strongest Pipeline-3 dynamic-stability reference
CF06   strongest Pipeline-3 structural / endpoint-energy reference
```

Pipeline 4 adds an independent AI design branch while preserving the central PEARL principle that no single score is treated as a universal measure of peptide quality.

---

## Biological and structural system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Interface:** chains `B–D`
- **Receptor chain:** `B`
- **Peptide chain:** `D`
- **Natural short reference:** `F0010 = IGERCQYRDLK`
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
| **4** | **MPNN_NEW_05** | **−53.67** | 6.43 |
| 5 | MPNN_NEW_01 | −46.41 | 4.66 |

`MPNN_NEW_05` is the endpoint-energy-favoured AI-derived candidate. The endpoint quantity is comparative and must not be interpreted as absolute binding free energy.

## 08g — ESMFold structural validation

`08g` validates externally generated ESMFold structures, confirms sequence identity and performs explicit comparison with the bound reference. It reports both confidence and geometric agreement rather than relying on pLDDT alone.

| Candidate | Mean pLDDT | Global Cα RMSD (Å) | Protected Cα RMSD after global fit (Å) | Mapped positions |
|---|---:|---:|---:|---:|
| **MPNN_NEW_01** | **72.56** | **4.49** | **3.67** | 11/11 |
| MPNN_NEW_05 | 67.27 | 5.07 | 3.83 | 11/11 |

Both sequence and structural-file QC passed. Within this comparison, `MPNN_NEW_01` is more confident and geometrically closer to the bound reference. For an isolated 11-residue peptide, pLDDT and RMSD remain structural descriptors—not affinity measurements.

## 08h — Solubility and developability screening

`08h` computes sequence-derived physicochemical descriptors with explicit provenance and supports validated import of authentic CamSol output. It never reports CamSol values unless a real run, version, mode and sequence-matched manifest are supplied.

The executed run was proxy-only:

| Candidate | GRAVY | Charge at pH 7 | Theoretical pI | CamSol |
|---|---:|---:|---:|---|
| **MPNN_NEW_01** | **−1.94** | +0.40 | 8.41 | not executed |
| MPNN_NEW_05 | −1.44 | +0.85 | 8.75 | not executed |

Both candidates had no heuristic alerts. `MPNN_NEW_01` is more hydrophilic according to the GRAVY proxy. These values are not experimental solubility measurements. The downstream metric contract therefore records:

```text
developability_metric_used   = proxy_gravy
developability_metric_source = sequence-derived Biopython GRAVY proxy
CamSol status                = not_executed
```

## 08i — Evidence integration and Bayesian-optimisation readiness

`08i` validates candidate identity and sequence hashes across 08f, 08g and 08h. It converts each domain into a direction-aware within-cohort utility:

```text
energy_utility_z
structural_utility_z
developability_utility_z
```

Structural pLDDT and RMSD are first combined into a single structural-domain signal so that structure is not double-weighted. With equal domain weights:

| Rank | Candidate | Composite proxy |
|---:|---|---:|
| **1** | **MPNN_NEW_01** | **0.333** |
| 2 | MPNN_NEW_05 | −0.333 |

Because only two candidates are available, the z-scores primarily encode ordering rather than reliable effect magnitude.

The weight-sensitivity scenarios produced:

```text
equal domains            → MPNN_NEW_01
structure priority       → MPNN_NEW_01
developability priority  → MPNN_NEW_01
energy priority          → tie for first
```

Bayesian optimisation was not executed:

```text
ENABLE_BOTORCH = False
BoTorch available = False
observations = 2
minimum workflow safeguard = 8
unevaluated candidate pool = absent
```

No Gaussian process, acquisition function, posterior or new sequence was fabricated.

## 08j — Final multi-objective decision support

`08j` consumes the validated 08i evidence contract, applies configurable equal domain weights, evaluates Pareto optimality and distinguishes outright victories from ties.

| Candidate | Final rank | Composite | Outright wins | Ties for first | Losses | Pareto-optimal |
|---|---:|---:|---:|---:|---:|---|
| **MPNN_NEW_01** | **1** | **0.333** | **3** | 1 | 0 | yes |
| MPNN_NEW_05 | 2 | −0.333 | 0 | 1 | 3 | yes |

The final evidence-aware interpretation is:

```text
MPNN_NEW_01 = best equal-weight exploratory balance
MPNN_NEW_05 = endpoint-energy-favoured alternative
both candidates = Pareto-optimal
```

Neither candidate dominates the other in every included domain. Pareto optimality does not establish universal biological superiority.

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

Fifty snapshots are selected uniformly from each 1 ns production trajectory. For every snapshot, the endpoint calculation evaluates complex, receptor and peptide energies from the same geometry using an AMBER protein force field and OBC2 implicit solvent.

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

### Candidate contract

`08h` reads the canonical manifest exported by 08f:

```text
08f_candidate_manifest.csv
```

with mandatory columns:

```text
candidate_id
sequence
```

The notebook normalises sequences, rejects empty or duplicated IDs, validates canonical amino acids and computes a SHA-256 hash for every sequence.

### Sequence-derived descriptors

Biopython `ProteinAnalysis` is used to calculate:

- molecular weight;
- theoretical isoelectric point;
- theoretical charge at pH 7;
- GRAVY;
- aromaticity;
- instability index.

Every such column uses the `proxy_` prefix. Heuristic alerts are kept separate and explicitly versioned as unvalidated triage rules.

The current full descriptor values include:

| Candidate | Molecular weight (Da) | pI | Charge pH 7 | GRAVY | Aromaticity | Instability index |
|---|---:|---:|---:|---:|---:|---:|
| MPNN_NEW_01 | 1316.42 | 8.406 | 0.398 | **−1.936** | 0.091 | −5.84 |
| MPNN_NEW_05 | 1351.51 | 8.753 | 0.847 | −1.436 | 0.091 | 15.03 |

### CamSol preparation

Even in proxy-only mode, 08h exports:

```text
08h_camsol_input.fasta
camsol_manifest_TEMPLATE.json
camsol_summary_TEMPLATE.csv
camsol_profiles_TEMPLATE.csv
```

The FASTA contains the exact candidate IDs, sequences and hashes submitted for a future real CamSol run.

### Validated CamSol import

Authentic CamSol import requires:

- a completed provenance manifest;
- reported CamSol version;
- `intrinsic` or `structure_corrected` mode;
- execution interface;
- run timestamp;
- raw output paths;
- one overall score per candidate;
- matching candidate ID and sequence hash;
- optional residue-level profile with complete indexing and residue identity.

If these requirements are not satisfied, 08h does not populate CamSol columns. The current run correctly reports:

```text
camsol_executed          = False
camsol_result_validated  = False
camsol_overall_score     = NaN
```

### Downstream selection rule

If a complete validated CamSol cohort is available, 08h can expose `camsol_overall_score` as the downstream developability signal. Otherwise it explicitly exposes `proxy_gravy`, with lower values treated as preferable. The metric name, provenance, direction and value are stored separately.

## Detailed 08i integration and readiness gates

### Input validation

08i requires all three upstream tables:

```text
08f_ProteinMPNN_cross_method_evidence.csv
08g_ESMFold_structural_validation_summary.csv
08h_solubility_developability_summary.csv
```

It verifies:

- existence of every required file;
- exact equality of candidate cohorts;
- unique candidate IDs;
- identical normalised sequences;
- matching SHA-256 sequence hashes;
- finite required metrics;
- completed reference comparisons;
- valid pLDDT status;
- explicit developability direction and provenance;
- internally consistent CamSol state.

### Domain utilities

The following raw metrics are converted into direction-aware within-cohort z-scores:

```text
lower endpoint energy  → higher energy utility
higher mean pLDDT      → higher pLDDT utility
lower global RMSD      → higher RMSD utility
08h-declared direction → higher developability utility
```

The pLDDT and RMSD utilities are averaged into `structural_utility_z`. The final three domains are therefore energy, structure and developability rather than four independently weighted columns.

With two candidates, the utilities are:

| Candidate | Energy utility | Structural utility | Developability utility |
|---|---:|---:|---:|
| MPNN_NEW_01 | −1.0 | **+1.0** | **+1.0** |
| MPNN_NEW_05 | **+1.0** | −1.0 | −1.0 |

### Weight scenarios

08i evaluates four transparent scenarios:

| Scenario | Energy | Structure | Developability | Outcome |
|---|---:|---:|---:|---|
| Equal domains | 1/3 | 1/3 | 1/3 | MPNN_NEW_01 |
| Energy priority | 0.50 | 0.25 | 0.25 | tie |
| Structure priority | 0.25 | 0.50 | 0.25 | MPNN_NEW_01 |
| Developability priority | 0.25 | 0.25 | 0.50 | MPNN_NEW_01 |

### Bayesian-optimisation gate

BoTorch availability alone would not authorise model training. The notebook requires all of the following:

- deliberate manual activation;
- installed BoTorch;
- at least the configured minimum number of observations;
- a non-empty unevaluated candidate pool;
- complete observed objectives;
- a separately reviewed feature and model specification.

The current analysis fails the first four readiness requirements. The recorded status is:

```text
not_run_readiness_requirements_not_met
```

## Detailed 08j final-decision workflow

08j consumes the harmonised 08i table rather than independently repeating upstream merges. It verifies sequence hashes, three-domain utility availability, developability provenance, CamSol consistency and Bayesian-optimisation status.

### Equal-weight composite

```text
final score =
    1/3 × energy_utility_z
  + 1/3 × structural_utility_z
  + 1/3 × developability_utility_z
```

The score is a decision-support heuristic. It is not an affinity or solubility prediction.

### Pareto analysis

A candidate is labelled Pareto-dominated only when another evaluated candidate is at least as good in all three utility domains and strictly better in at least one. Because `MPNN_NEW_05` is better in energy while `MPNN_NEW_01` is better in structure and developability, both candidates are Pareto-optimal.

### Robustness accounting

The notebook analyses every 08i weight scenario and distinguishes:

- outright wins;
- ties for first;
- losses;
- total scenarios ranked first.

This avoids the earlier ambiguity in which a tied rank could be counted as a full independent victory.

### Final QC

The executed 08j run passed 13 checks covering:

1. existence of all required 08i inputs;
2. candidate-ID uniqueness;
3. sequence-hash validation;
4. presence of all three domain utilities;
5. weights summing to one;
6. complete ranks;
7. complete Pareto status;
8. complete sensitivity outcomes;
9. consistent win/tie/loss accounting;
10. explicit developability provenance;
11. consistent CamSol state;
12. explicit Bayesian-optimisation status;
13. absence of an unreported sequence proposal.

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
| CamSol | not executed in the current run | experimental solubility |
| BoTorch | not executed; readiness only | validated posterior or proposal |

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

- **CamSol:** authentic results can be imported by 08h with validated provenance. CamSol was not executed in the current run.
- **BoTorch:** reserved for a future expanded observed design set and an explicit unevaluated candidate pool. It was not installed or executed in the current run.

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

- harmonised observed evidence;
- exploratory ranking;
- weight-sensitivity table;
- candidate-proposal status;
- BoTorch-readiness table;
- evidence-integration figure;
- `08i_QC.csv`, provenance JSON and report.

### 08j

- final multi-objective ranking;
- scenario-level outcomes;
- candidate robustness summary;
- evidence-status table;
- final multi-panel figure;
- `08j_QC.csv`, provenance JSON and report.

---

# Interpretation and limitations

1. ESM-2 and ProteinMPNN are not affinity predictors.
2. FoldX and Rosetta scores are model-dependent approximations.
3. The 1 ns MD trajectories are useful for comparative screening but do not establish full conformational convergence.
4. The endpoint calculation is a single-trajectory MM/GBSA-like proxy, not absolute binding free energy.
5. ESMFold confidence and RMSD do not demonstrate receptor binding.
6. GRAVY, charge, pI and related descriptors are developability proxies, not measured solubility.
7. CamSol was not executed; no CamSol-derived score is claimed.
8. Bayesian optimisation was not executed because two observations and no candidate pool do not support a defensible campaign.
9. Within-cohort z-scores for two candidates mostly encode ordering.
10. Equal domain weights are transparent but subjective; sensitivity scenarios must accompany the final ranking.
11. Pareto optimality means only that no evaluated candidate dominates another in every included domain.
12. Experimental validation remains required.

Experimental studies would be needed to determine binding, affinity, inhibition, selectivity, solubility, stability, toxicity and cellular activity.

---

# Final Pipeline 4 conclusion

Pipeline 4 successfully adds an independent ESM-2 and ProteinMPNN design branch to PEARL and carries the resulting candidates through layered structural, dynamic, energetic and decision-support analyses.

The final cross-method picture is:

```text
Established references
F0010  natural / miniaturised reference
CF02   strongest dynamic-stability reference
CF06   strongest structural / endpoint-energy reference

ProteinMPNN candidates
MPNN_NEW_01  best balanced exploratory candidate
MPNN_NEW_05  endpoint-energy-favoured alternative
```

The final ranking among the two advanced AI-derived candidates is:

```text
1. MPNN_NEW_01  TGPRNQYRDLP   composite  0.333
2. MPNN_NEW_05  IGPRHQYRDLP   composite −0.333
```

This ordering reflects a transparent equal-weight trade-off:

- `MPNN_NEW_01` is favoured by short-timescale MD, ESMFold structural validation and the GRAVY proxy;
- `MPNN_NEW_05` is favoured by FoldX, Rosetta and endpoint energy;
- both candidates are Pareto-optimal;
- the energy-priority scenario produces a tie;
- CamSol and Bayesian optimisation remain future extensions rather than completed analyses.

The strongest methodological result is the complementarity of the rankings. Pipeline 4 therefore reinforces the PEARL principle that candidate prioritisation should rely on converging evidence, explicit provenance and visible uncertainty—not on a single AI, structural, dynamic, energetic or composite score.

All final candidates remain computational hypotheses. Longer replicated simulations, more rigorous free-energy calculations, authentic CamSol results where appropriate and experimental validation are required before quantitative biological claims can be made.
