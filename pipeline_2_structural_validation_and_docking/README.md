# PEARL Pipeline 2 — Structural Validation, CLEAR Optimization and Peptide Docking

This directory contains the second-stage computational development of the
PEARL project.
The notebooks were developed after the initial PEARL prototype in order to
explore alternative interface-derived peptides, improve candidate generation,
introduce a CLEAR-inspired counterfactual optimization workflow, and perform
more extensive structural and energetic validation using FoldX and Rosetta
FlexPepDock.
This directory therefore represents a broader experimental and methodological
pipeline rather than a simple numerical continuation of Pipeline 1.

This second pipeline contains:

02c_Hotspot_Centered_Peptide_Library.ipynb
02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb
03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb
03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb
04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb
04d_CLEAR_Peptide_Oracle_Training.ipynb
04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb
05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb
05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb
---
## Relationship with Pipeline 1


## Pipeline 1 provides the first implementation of the workflow:

EGFR structure
    ↓
interface identification
    ↓
biological-interface validation
    ↓
FoldX alanine scanning
    ↓
energetic-hotspot identification
    ↓
seed-peptide extraction
    ↓
candidate generation
    ↓
pre-docking ranking

## Pipeline 2 extends this work through:

interface subgraph analysis
    ↓
hotspot-centred and short-peptide design
    ↓
structural and energetic ranking
    ↓
guided peptide docking
    ↓
local peptide-variant dataset construction
    ↓
peptide-oracle training
    ↓
CLEAR-inspired counterfactual optimization
    ↓
FoldX structural validation
    ↓
Rosetta FlexPepDock refinement
    ↓
integrated candidate evaluation

Some notebooks revisit or improve analyses introduced in Pipeline 1.
For this reason, the notebook prefixes should not be interpreted as one
strictly linear execution sequence.

⸻

Biological system

* Target: Epidermal Growth Factor Receptor, EGFR
* Reference structure: PDB 3NJP
* Initial interface studied: chains B–D
* Reference peptide source: contiguous regions extracted from the selected
    protein–protein interface
* Main objective: identify and structurally prioritize peptide candidates
    capable of reproducing important interface interactions

The analyses remain computational and do not constitute experimental evidence
of peptide binding or inhibition.

⸻

## 02c_Hotspot_Centered_Peptide_Library.ipynb

Constructs a library of peptide sequences centred around important interface
hotspots.

Main operations include:

* selection of energetic or structurally central hotspot residues;
* construction of peptide windows around those residues;
* generation of peptides with different lengths and boundaries;
* annotation of hotspot coverage;
* comparison with the original seed peptide;
* creation of a candidate library for subsequent ranking.

The resulting sequences are candidate peptide hypotheses and are not assumed to
be experimentally active.

⸻

## 02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb

Ranks shorter interface-derived peptides using structural and energetic
criteria.

Possible criteria include:

* number of retained interface residues;
* number of retained energetic hotspots;
* contact coverage;
* graph-based importance;
* FoldX-derived energetic information;
* peptide length;
* structural compactness;
* sequence and physicochemical properties.

The purpose is to identify shorter candidates that preserve a substantial part
of the original interface signal.

⸻

## 03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb

Extends and corrects the contiguous-window workflow by explicitly integrating
energetic-hotspot information.

Main operations include:

* ranking contiguous peptide windows;
* evaluating hotspot retention;
* selecting the reference seed;
* generating conservative sequence variants;
* combining sequence, structural and energetic criteria;
* exporting prioritized candidates.

The FIXED suffix indicates that this notebook contains corrections or
stabilized logic relative to earlier versions.

⸻

## 03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb

Prepares selected short peptides for receptor–peptide structural evaluation.

Main operations may include:

* extraction or construction of receptor–peptide complexes;
* assignment of receptor and peptide chains;
* structural consistency checks;
* preparation of docking inputs;
* generation of guided docking configurations;
* definition of restraints or starting peptide positions;
* execution or preparation of FoldX and Rosetta calculations.

This notebook connects short-peptide selection with explicit three-dimensional
structural evaluation.

⸻

## 03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb

Performs deeper Rosetta FlexPepDock refinement for the top three selected
short-peptide candidates.

The N20 designation indicates that multiple structural models or decoys are
generated for each candidate, typically with:

nstruct = 20

Main operations include:

* selection of the top three peptide complexes;
* high-resolution peptide–protein refinement;
* generation of multiple docking decoys;
* parsing of Rosetta score files;
* comparison of the best-scoring models;
* inspection of score distributions and structural convergence.

The best numerical Rosetta score should not be interpreted alone as proof of
binding.

⸻

## 04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb

Constructs a structured local peptide-variant dataset for the CLEAR-inspired
workflow.

Main operations include:

* definition of the reference seed;
* generation or collection of local peptide variants;
* enforcement of mutation and locality constraints;
* assignment of candidate and chunk identifiers;
* preparation of FoldX runs;
* parsing of FoldX difference-energy files;
* association of peptide sequences with energetic labels;
* construction of a consolidated machine-learning dataset;
* validation of completed, missing or failed calculations.

In this pipeline, CLEAR principles are adapted to peptide sequences by requiring
counterfactual candidates to remain local, constrained and interpretable with
respect to the reference seed.

This is not identical to the original graph-based CLEAR implementation for
small molecular graphs.

⸻

## 04d_CLEAR_Peptide_Oracle_Training.ipynb

Trains a predictive peptide oracle using the local variant dataset.

The oracle is intended to approximate an energetic or structural target derived
from the computed peptide variants.

Main operations may include:

* loading and cleaning the variant dataset;
* encoding peptide sequences;
* defining training, validation and test partitions;
* training a predictive model;
* monitoring loss and validation performance;
* evaluating prediction errors;
* saving the trained model and preprocessing objects;
* determining whether the model is sufficiently reliable for optimization.

The oracle is a computational surrogate and its predictions remain dependent on
the quality and coverage of the training dataset.

⸻

## 04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb

Uses the trained peptide oracle to generate and prioritize local
counterfactual peptide candidates.

Main objectives include:

* starting from the reference seed;
* proposing a limited number of sequence substitutions;
* preserving protected or hotspot positions where required;
* optimizing the predicted target property;
* penalizing excessive or non-local sequence changes;
* generating interpretable counterfactual sequences;
* comparing optimized candidates with the original peptide;
* exporting candidates for FoldX and docking validation.

A counterfactual candidate should answer a question such as:

What is the smallest admissible sequence modification predicted to improve
the selected energetic or structural property?

The oracle prediction is not treated as final validation. Counterfactual
candidates must be re-evaluated using explicit structural tools.

⸻

## 05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb

Validates CLEAR-generated counterfactual peptides using explicit structural and
FoldX calculations.

Main operations may include:

* loading counterfactual sequences generated in Notebook 04e;
* mapping mutations onto the peptide structure;
* constructing receptor–counterfactual complexes;
* structural integrity checks;
* FoldX complex-energy evaluation;
* comparison with the original seed;
* comparison with conventionally generated candidates;
* rejection of structurally invalid or energetically unfavourable candidates;
* prioritization of candidates for Rosetta refinement.

This notebook acts as the first structure-based validation layer after
oracle-guided counterfactual optimization.

⸻

## 05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb

Performs Rosetta FlexPepDock refinement of the counterfactual candidates that
passed the FoldX and structural-validation stage.

Main operations include:

* selection of validated CLEAR counterfactuals;
* preparation of FlexPepDock runs;
* generation of multiple refined receptor–peptide structures;
* parsing of Rosetta score files;
* comparison with the reference seed and non-CLEAR candidates;
* evaluation of docking-score distributions;
* inspection of final peptide poses;
* prioritization of structurally plausible counterfactual candidates.

This notebook provides a more computationally expensive validation step and
should normally be run after Notebook 05c.

⸻


## Recommended execution logic

Pipeline 2 contains alternative and complementary branches. It should therefore
not always be executed as one uninterrupted sequence.

## Branch 1 — Short hotspot-centred peptides

02_Interface_Subgraph_Search
    ↓
02c_Hotspot_Centered_Peptide_Library
    ↓
02d_Short_Peptide_Structural_and_Energetic_Ranking
    ↓
03c_Short_Peptide_Structural_Preparation_and_Guided_Docking
    ↓
03d_Top3_FlexPepDock_Deep_Refinement_N20

## Branch 2 — Contiguous seed and generated peptide variants

03_Contiguous_Peptide_Window_Diffusion
        or
03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED
    ↓
04b_Candidate_Validation_PreDocking_EnergeticHotspots
    ↓
05b_Structural_Preparation_Seed_and_Candidates_FoldX
    ↓
06b_Guided_Peptide_Docking_and_Interface_Evaluation

For the corrected hotspot-aware workflow, Notebook 03b should generally be
preferred over Notebook 03.

## Branch 3 — CLEAR-inspired counterfactual peptide optimization

04c_CLEAR_Local_Peptide_Variant_Dataset
    ↓
04d_CLEAR_Peptide_Oracle_Training
    ↓
04e_CLEAR_Peptide_Counterfactual_Optimization
    ↓
05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation
    ↓
05d_CLEAR_Counterfactual_FlexPepDock_Refinement

The three branches can ultimately be compared through structural, energetic and
docking-based evaluation.

⸻

## External software

# FoldX

FoldX is used for:

* energetic hotspot calculations;
* mutation-energy estimation;
* receptor–peptide interaction analysis;
* comparison of seed and candidate complexes;
* validation of CLEAR-generated counterfactuals.

FoldX must be installed separately and its executable path may need to be
configured manually.

⸻

# Rosetta FlexPepDock

Rosetta FlexPepDock is used for high-resolution peptide–protein refinement.

A valid Rosetta installation must provide:

* the FlexPepDocking executable;
* the Rosetta database;
* binaries compatible with the local operating system.

The notebooks may either run Rosetta directly or generate commands and scripts
for manual execution.

⸻

## Expected outputs

Depending on the selected branch, the pipeline may generate:

* interface-subgraph tables;
* hotspot-centred peptide libraries;
* ranked short-peptide candidates;
* contiguous peptide-window rankings;
* conservative sequence variants;
* pre-docking validation tables;
* CLEAR local variant datasets;
* trained peptide-oracle models;
* counterfactual peptide candidates;
* FoldX input structures and energy files;
* Rosetta docking scripts;
* refined PDB structures;
* Rosetta score files;
* integrated candidate rankings;
* diagnostic plots and CSV reports.

Generated data may be stored in directories such as:

outputs/
runs/
foldx_runs/
rosetta_runs/

Large temporary files, external software and large docking-output collections
should normally not be committed to the GitHub repository.

⸻

## Interpretation and limitations

This pipeline produces computational hypotheses.

The following points should be considered:

* interface contacts do not alone demonstrate biological relevance;
* a predicted hotspot is not necessarily an experimentally confirmed hotspot;
* FoldX energies are approximate computational estimates;
* the peptide oracle is limited by its training dataset;
* counterfactual improvement refers to the modelled target, not necessarily to
    experimental affinity;
* a favourable Rosetta score is not a binding-affinity measurement;
* docking results depend on the starting pose and sampling procedure;
* small score differences may not be biologically meaningful;
* peptide stability, solubility, selectivity and cellular activity require
    additional evaluation;
* experimental validation is ultimately required.

⸻

Notebook naming

The notebook names are preserved as originally developed.

Prefixes such as 02c, 03d, 04e and 05d reflect the chronological and
experimental development of the work. They do not necessarily indicate a
single mandatory linear sequence.

The execution paths described in this README should be used to understand the
relationship between notebooks.

⸻

Project status

* Stage: second PEARL computational development phase
* Scope: peptide miniaturization, candidate generation, CLEAR-inspired
    optimization and structural validation
* Validation level: computational and non-experimental
* Intended use: research, methodological development and academic
    presentation

## Struttura della cartella
 
pipeline_2_structural_validation_and_docking/

├── README.md 
├── 02c_Hotspot_Centered_Peptide_Library.ipynb 
├── 02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb 
├── 03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb 
├── 03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb 
├── 04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb 
├── 04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb 
├── 04d_CLEAR_Peptide_Oracle_Training.ipynb 
├── 04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb 
├── 05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb 
├── 05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb 

