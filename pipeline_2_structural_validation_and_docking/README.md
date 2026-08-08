# PEARL Pipeline 2 — Structural Validation, CLEAR Optimization and Peptide Docking

This directory contains the second-stage computational development of the PEARL project.

The notebooks included here were developed after the initial PEARL prototype in order to explore two complementary strategies:

1. the design and structural evaluation of short hotspot-centred peptides;
2. the generation and validation of CLEAR-inspired counterfactual peptide candidates.

Pipeline 2 is therefore not a simple numerical continuation of Pipeline 1. It represents a separate stage of methodological development focused on peptide miniaturization, local sequence optimization, FoldX evaluation, Rosetta FlexPepDock refinement and interpretable adjacent-residue pair scoring.

---

## Relationship with Pipeline 1

The initial PEARL prototype is stored in `pipeline_1_initial_prototype/`.

Pipeline 1 provides the first implementation of the workflow:

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
    pre-docking prioritization

Pipeline 2 starts from selected structural and sequence information obtained during the first stage and extends the project through two complementary computational branches.

### Branch 1 — Short hotspot-centred peptides

    hotspot-centred peptide library
        ↓
    structural and energetic ranking
        ↓
    guided peptide docking
        ↓
    Rosetta FlexPepDock refinement

### Branch 2 — CLEAR-inspired peptide optimization

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
    adjacent amino-acid pair scoring and interpretation

---

## Biological system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Initial interface studied:** chains `B–D`
- **Reference peptide source:** contiguous regions extracted from the selected protein–protein interface
- **Main objective:** identify and structurally prioritize peptide candidates capable of retaining important interface interactions

The analyses contained in this directory are computational and do not constitute experimental evidence of peptide binding, inhibition or biological activity.

---

## Notebooks included

### Short-peptide branch

- `02c_Hotspot_Centered_Peptide_Library.ipynb`
- `02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb`
- `03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb`
- `03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb`

### CLEAR-inspired branch

- `04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb`
- `04d_CLEAR_Peptide_Oracle_Training.ipynb`
- `04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb`
- `05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb`
- `05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb`
- `05e_CLEAR_Adjacent_Amino_Acid_Pair_Energy_Scoring.ipynb`

The notebook names are preserved as originally developed.

Prefixes such as `02c`, `03d`, `04e` and `05d` reflect the chronological and experimental development of the work. They do not indicate one single mandatory linear sequence across the whole directory.

---

# Branch 1 — Short Hotspot-Centred Peptide Design

## `02c_Hotspot_Centered_Peptide_Library.ipynb`

This notebook constructs a library of short peptide sequences centred around important interface hotspots.

Main operations include:

- selection of structurally or energetically important interface residues;
- definition of peptide windows around hotspot positions;
- generation of peptides with different lengths and boundaries;
- annotation of hotspot coverage;
- comparison with the original interface-derived peptide;
- creation of a candidate library for subsequent ranking.

The resulting sequences are computational peptide hypotheses and are not assumed to be experimentally active.

---

## `02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb`

This notebook ranks the short hotspot-centred peptides using structural and energetic criteria.

Possible ranking criteria include:

- retained interface residues;
- retained hotspot positions;
- contact coverage;
- graph-based residue importance;
- energetic contribution;
- peptide length;
- structural coherence;
- physicochemical properties.

The purpose is to identify shorter candidates that preserve a relevant fraction of the original interface signal.

---

## `03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb`

This notebook prepares selected short peptides for explicit receptor–peptide structural evaluation.

Main operations include:

- loading the receptor structure;
- preparing receptor–peptide complexes;
- assigning receptor and peptide chains;
- checking structural consistency;
- generating docking input structures;
- defining guided docking configurations;
- preparing FoldX or Rosetta calculations;
- organizing candidate-specific run directories and outputs.

This notebook connects peptide ranking with three-dimensional structural evaluation.

---

## `03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb`

This notebook performs deeper Rosetta FlexPepDock refinement for the top three selected short-peptide candidates.

The `N20` designation indicates that multiple structural models are generated for each candidate, typically using `nstruct = 20`.

Main operations include:

- selection of the top three peptide complexes;
- high-resolution peptide–protein refinement;
- generation of multiple structural decoys;
- parsing of Rosetta score files;
- comparison of the best-scoring models;
- inspection of score distributions;
- evaluation of structural convergence.

The best Rosetta score should not be interpreted alone as proof of peptide binding.

---

# Branch 2 — CLEAR-Inspired Peptide Optimization

## `04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb`

This notebook constructs a structured local peptide-variant dataset around the reference seed sequence.

Main operations include:

- definition of the reference seed;
- generation or collection of local peptide variants;
- enforcement of sequence-locality constraints;
- definition of mutable and protected peptide positions;
- assignment of candidate identifiers;
- organization of variants into chunks;
- preparation of FoldX calculations;
- parsing of energetic output files;
- association of peptide sequences with computed labels;
- construction of a consolidated machine-learning dataset;
- validation of completed, missing or failed calculations.

In this pipeline, CLEAR principles are adapted to peptide sequences by requiring candidate variants to remain local, constrained and interpretable with respect to the reference peptide.

This workflow is inspired by the general logic of CLEAR, but it is not identical to the original graph-counterfactual framework for small molecular graphs.

---

## `04d_CLEAR_Peptide_Oracle_Training.ipynb`

This notebook trains a predictive peptide oracle using the local peptide-variant dataset.

The oracle is intended to approximate a structural or energetic target associated with the generated peptide variants.

Main operations include:

- loading and cleaning the peptide dataset;
- sequence encoding;
- definition of training, validation and test partitions;
- model training;
- monitoring of training and validation loss;
- prediction-error analysis;
- model selection;
- saving the trained oracle and preprocessing objects;
- assessment of whether the oracle is sufficiently reliable for optimization.

The oracle is a computational surrogate model. Its predictions depend on the quality, size and coverage of the underlying dataset.

---

## `04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb`

This notebook uses the trained peptide oracle to generate local counterfactual peptide candidates.

The objective is to identify limited and interpretable sequence modifications that improve the target predicted by the oracle.

Main operations include:

- starting from the reference peptide;
- proposing a limited number of amino-acid substitutions;
- preserving protected or hotspot positions when required;
- enforcing locality and mutation constraints;
- optimizing the predicted target property;
- penalizing excessive sequence changes;
- generating interpretable counterfactual candidates;
- comparing optimized sequences with the original peptide;
- exporting candidates for explicit structural validation.

A counterfactual candidate can be interpreted as an answer to the question:

> What is the smallest admissible sequence modification predicted to improve the selected structural or energetic property?

Oracle predictions are not treated as final evidence. Counterfactual candidates must be validated using explicit structural calculations.

---

## `05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb`

This notebook validates CLEAR-generated counterfactual peptides using FoldX and explicit structural checks.

Main operations include:

- loading counterfactual sequences generated in Notebook `04e`;
- mapping mutations onto the peptide structure;
- constructing receptor–counterfactual complexes;
- checking structural integrity;
- preparing FoldX calculations;
- evaluating receptor–peptide interaction energies;
- comparing counterfactuals with the original reference peptide;
- rejecting structurally invalid or energetically unfavourable candidates;
- prioritizing candidates for Rosetta refinement.

This notebook represents the first structure-based validation stage after oracle-guided optimization.

---

## `05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb`

This notebook performs Rosetta FlexPepDock refinement of the CLEAR counterfactual candidates that passed the previous structural and FoldX validation stage.

Main operations include:

- selection of validated counterfactual peptides;
- preparation of Rosetta FlexPepDock runs;
- generation of multiple refined receptor–peptide structures;
- parsing of Rosetta score files;
- comparison with the reference peptide;
- analysis of score distributions;
- inspection of final peptide poses;
- prioritization of structurally plausible counterfactual candidates.

This stage provides a more computationally expensive validation layer and should normally be performed after Notebook `05c`.

---

## `05e_CLEAR_Adjacent_Amino_Acid_Pair_Energy_Scoring.ipynb`

This notebook implements an interpretable scoring function for adjacent amino-acid pairs in the peptide sequence.

For a peptide sequence of length `L`, the notebook evaluates the `L - 1` adjacent pairs and computes a position-specific surrogate cost:

    (a1,a2), (a2,a3), ..., (aL-1,aL)

Main operations include:

- extraction of all adjacent amino-acid pairs from the local 04c variant dataset;
- estimation of position-specific pair statistics;
- shrinkage regularization for rarely observed pairs;
- conversion of the local target signal into an empirical pair cost;
- comparison of each observed pair with alternative combinations allowed by the CLEAR positional constraints;
- calculation of total and position-wise pair costs for F0010 and the final CLEAR candidates;
- decomposition of sequence changes relative to the seed peptide;
- integration of pair scores with FoldX and Rosetta results;
- computation of correlations with FoldX `ΔΔG`, FlexPepDock scores and the final structural ranking;
- export of reusable pair-cost tables and the function `adjacent_pair_energy_score(sequence)`.

The notebook supports interpretation of why specific substitutions are preferred by the local surrogate model. For example, a single mutation may alter two adjacent pairs simultaneously and therefore accumulate multiple local contributions.

The resulting value is an **empirical, position-specific surrogate energetic cost**. It is not an absolute physical energy in kcal/mol and does not replace FoldX, Rosetta or experimental validation.

---

## Recommended execution logic

Pipeline 2 contains two complementary branches and should not be interpreted as one uninterrupted notebook sequence.

### Branch 1 — Short hotspot-centred peptides

    02c_Hotspot_Centered_Peptide_Library
        ↓
    02d_Short_Peptide_Structural_and_Energetic_Ranking
        ↓
    03c_Short_Peptide_Structural_Preparation_and_Guided_Docking
        ↓
    03d_Top3_FlexPepDock_Deep_Refinement_N20

### Branch 2 — CLEAR-inspired counterfactual peptide optimization

    04c_CLEAR_Local_Peptide_Variant_Dataset
        ↓
    04d_CLEAR_Peptide_Oracle_Training
        ↓
    04e_CLEAR_Peptide_Counterfactual_Optimization
        ↓
    05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation
        ↓
    05d_CLEAR_Counterfactual_FlexPepDock_Refinement
        ↓
    05e_CLEAR_Adjacent_Amino_Acid_Pair_Energy_Scoring

The two branches provide complementary strategies for peptide design and structural prioritization.

The first branch investigates peptide miniaturization around interface hotspots.

The second branch investigates local, interpretable and model-guided peptide optimization.

---

## External software

### FoldX

FoldX is used for:

- structural preparation;
- receptor–peptide complex analysis;
- interaction-energy estimation;
- comparison of reference and candidate complexes;
- validation of CLEAR-generated counterfactual peptides.

FoldX must be installed separately and its executable path may need to be configured manually.

Depending on the notebook and execution environment, FoldX may be:

- executed directly from Python;
- invoked through generated shell commands;
- run manually outside the notebook;
- parsed after completion.

Typical output files may include:

- `Interaction_<candidate>_AC.fxout`
- `Indiv_energies_<candidate>_AC.fxout`

### Rosetta FlexPepDock

Rosetta FlexPepDock is used for high-resolution peptide–protein refinement.

A valid Rosetta installation must provide:

- the `FlexPepDocking` executable;
- the Rosetta database;
- binaries compatible with the local operating system.

The notebooks may either execute Rosetta directly or generate scripts and commands for manual execution.

Typical refinement options may include:

- `-pep_refine`
- `-ex1`
- `-ex2aro`
- `-use_input_sc`
- `-nstruct`

---

## Python dependencies

The notebooks use standard scientific and structural-bioinformatics tools, including:

- Python 3;
- Jupyter Notebook;
- NumPy;
- pandas;
- Matplotlib;
- Biopython;
- scikit-learn;
- pathlib;
- subprocess;
- regular-expression utilities.

Additional dependencies may be required according to the notebook and local environment.

---

## Expected outputs

Depending on the selected branch, the pipeline may generate:

- hotspot-centred peptide libraries;
- ranked short-peptide candidates;
- prepared receptor–peptide complexes;
- guided-docking input files;
- CLEAR local peptide-variant datasets;
- trained peptide-oracle models;
- counterfactual peptide candidates;
- FoldX input structures;
- FoldX interaction-energy files;
- Rosetta FlexPepDock scripts;
- refined PDB structures;
- Rosetta score files;
- structural and energetic candidate rankings;
- position-specific adjacent-pair cost matrices;
- candidate pairwise scores and pair-level changes relative to F0010;
- rankings of alternative amino-acid pairs allowed by CLEAR constraints;
- integrated pair-score, FoldX and Rosetta comparison tables;
- diagnostic plots;
- CSV reports.

Generated files may be stored in directories such as:

- `outputs/`
- `runs/`
- `foldx_runs/`
- `rosetta_runs/`

Large temporary files, external software installations and extensive docking-output collections should normally not be committed to the repository.

---

## Interpretation and limitations

This pipeline produces computational hypotheses.

The following limitations should be considered:

- a predicted hotspot is not necessarily an experimentally confirmed hotspot;
- a valid peptide sequence is not necessarily stable or soluble;
- FoldX energies are approximate computational estimates;
- the peptide oracle is limited by its training dataset;
- counterfactual improvement refers to the modelled target;
- a favourable oracle prediction does not guarantee improved binding;
- a favourable Rosetta score is not a binding-affinity measurement;
- the adjacent-pair score is an empirical surrogate derived from the local 04c dataset;
- adjacent-pair costs are position-specific and are not absolute physical energies;
- the pairwise analysis is not independent of the oracle target and should be used mainly for interpretation;
- docking results depend on the starting pose and sampling protocol;
- small score differences may not be biologically meaningful;
- peptide selectivity, stability, toxicity and cellular activity are not established;
- molecular-dynamics analysis may be required for further validation;
- experimental validation is ultimately required.

The final candidates should therefore be interpreted as ranked structural hypotheses rather than confirmed inhibitors.

---

## Directory structure

    pipeline_2_structural_validation_and_docking/
    ├── README.md
    ├── 02c_Hotspot_Centered_Peptide_Library.ipynb
    ├── 02d_Short_Peptide_Structural_and_Energetic_Ranking.ipynb
    ├── 03c_Short_Peptide_Structural_Preparation_and_Guided_Docking.ipynb
    ├── 03d_Top3_FlexPepDock_Deep_Refinement_N20.ipynb
    ├── 04c_CLEAR_Local_Peptide_Variant_Dataset.ipynb
    ├── 04d_CLEAR_Peptide_Oracle_Training.ipynb
    ├── 04e_CLEAR_Peptide_Counterfactual_Optimization.ipynb
    ├── 05c_CLEAR_Counterfactual_FoldX_and_Structural_Validation.ipynb
    ├── 05d_CLEAR_Counterfactual_FlexPepDock_Refinement.ipynb
    └── 05e_CLEAR_Adjacent_Amino_Acid_Pair_Energy_Scoring.ipynb

---

## Project status

- **Stage:** second PEARL computational development phase
- **Scope:** peptide miniaturization, CLEAR-inspired optimization and structural validation
- **Validation level:** computational and non-experimental
- **Purpose:** research, methodological development and academic presentation
  
## Optional VAE extension

An experimental VAE-based extension is provided in:

`vae_extension/`

This branch implements a true latent-space counterfactual workflow:

04c local peptide dataset
→ 06a VAE training
→ 06b latent-space validation
→ 06c latent counterfactual optimization
→ 06d comparison with Direct CLEAR

The VAE extension is complementary to the consolidated
Direct CLEAR workflow and does not replace notebooks 04c–05e.

Direct CLEAR performs constrained peptide optimization directly
in sequence/categorical space, whereas the VAE extension learns
a continuous latent representation and performs counterfactual
optimization in that latent space.


