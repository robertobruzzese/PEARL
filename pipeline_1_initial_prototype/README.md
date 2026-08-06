# PEARL Pipeline 1 — Initial Prototype

This directory contains the first computational prototype developed for the PEARL project.

The purpose of this pipeline is to start from an EGFR crystallographic structure, identify a candidate protein–protein interface, estimate structurally and energetically important interface residues, extract a contiguous seed peptide, generate sequence variants, and prioritize peptide candidates before structural docking.

Pipeline 1 should be interpreted as the initial end-to-end implementation of the PEARL workflow. The notebooks in this directory form one sequential pipeline and should be executed in the order reported below.

---

## Pipeline overview

    EGFR PDB structure
        ↓
    chain and interface analysis
        ↓
    biological and structural interface validation
        ↓
    FoldX alanine scanning and hotspot identification
        ↓
    contiguous seed-peptide extraction
        ↓
    local peptide-variant generation
        ↓
    pre-docking candidate validation and ranking

---

## Biological system

- **Target:** Epidermal Growth Factor Receptor, EGFR
- **Reference structure:** PDB `3NJP`
- **Initial interface selected:** chains `B–D`
- **Initial heavy-atom contact cutoff:** 4.5 Å
- **Reference peptide source:** a contiguous region extracted from the selected protein–protein interface
- **Main output:** a ranked collection of peptide candidates derived from an interface seed sequence

The `B–D` interface was initially selected by comparing the number of inter-chain structural contacts. This choice was subsequently examined in the biological-interface validation notebook.

The analyses contained in this directory are computational and do not constitute experimental evidence of peptide binding, inhibition or biological activity.

---

## Notebooks included

- `01_EGFR_PDB_to_Interface_Graph.ipynb`
- `01B_Biological_Interface_Validation.ipynb`
- `02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb`
- `03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb`
- `04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb`

Notebook `01B` is an autonomous validation step placed between Notebook `01` and Notebook `02b`. It should not be interpreted merely as an alternative version of Notebook `01`.

---

## Recommended execution order

    01_EGFR_PDB_to_Interface_Graph
        ↓
    01B_Biological_Interface_Validation
        ↓
    02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED
        ↓
    03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED
        ↓
    04b_Candidate_Validation_PreDocking_EnergeticHotspots

---

# Notebook descriptions

## `01_EGFR_PDB_to_Interface_Graph.ipynb`

This notebook downloads and parses the EGFR structure, identifies the available chains, calculates inter-chain contacts and constructs a residue-level interface graph.

Main operations include:

- retrieval and parsing of PDB structure `3NJP`;
- inspection of models, chains and residues;
- exclusion of hydrogen atoms from contact analysis;
- detection of heavy-atom contacts using a 4.5 Å cutoff;
- comparison of all chain-pair interfaces;
- initial selection of the `B–D` interface;
- construction of a residue-level interface graph;
- calculation of degree and betweenness centrality;
- identification of structurally central interface residues;
- export of contact tables and graph-related statistics.

The number of structural contacts is used as an initial geometric criterion. It is not, by itself, sufficient to establish that an interface is biologically relevant.

---

## `01B_Biological_Interface_Validation.ipynb`

This notebook provides an additional biological and structural validation step for the interface selected in Notebook `01`.

Its purpose is to determine whether the interface selected from structural contact analysis is also compatible with:

- the biological assembly of the crystallographic structure;
- the known organization of the EGFR complex;
- chain identity and symmetry;
- structural annotations associated with the PDB entry;
- the biological plausibility of the selected chain pair.

This notebook should be interpreted as a validation and interpretation stage rather than as an independent peptide-design procedure.

Its role is to reduce the risk of carrying an interface forward solely because it contains the largest number of geometric contacts.

---

## `02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb`

This notebook estimates the energetic contribution of interface residues through computational alanine scanning using FoldX.

Main operations include:

- preparation of the selected interface structure;
- definition of interface residues to be scanned;
- mutation of selected residues to alanine;
- generation of FoldX `BuildModel` inputs;
- execution or preparation of FoldX calculations;
- parsing of mutation-induced energy differences;
- identification of candidate energetic hotspots;
- integration of FoldX, contact and graph-based information;
- construction of ranked interface-residue tables;
- identification of contiguous regions enriched in important residues.

A residue is considered a candidate energetic hotspot when its mutation to alanine is predicted to destabilize the interface by a sufficiently large amount.

In this prototype, a positive mutation-induced energy change indicates that the native residue is predicted to contribute favourably to interface stability.

FoldX values are computational estimates and should not be interpreted as experimental binding free energies.

---

## `03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb`

This notebook searches the selected interface chain for contiguous peptide windows that retain important interface residues and energetic hotspots.

Main operations include:

- construction of contiguous peptide windows;
- comparison of alternative window positions;
- ranking using structural, graph-based and energetic information;
- selection of a reference seed peptide;
- annotation of interface and hotspot positions within the seed;
- generation of conservative local sequence variants;
- preservation of important residues where required;
- candidate deduplication;
- sequence-level and pre-docking scoring;
- export of the seed and generated candidates.

The initial prototype selected a 30-residue seed peptide from chain `D`:

    YIEALDKYACNCVVGYIGERCQYRDLKWWE

The generated candidates are local sequence variants of this seed. They should be interpreted as computational hypotheses rather than validated peptide binders.

The `FIXED` suffix indicates that this notebook contains corrections and stabilized logic relative to earlier development versions.

---

## `04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb`

This notebook performs sequence-level quality control, validation and pre-docking ranking of the generated peptide candidates.

Main operations include:

- amino-acid sequence validation;
- peptide-length consistency checks;
- mutation counting;
- candidate deduplication;
- comparison with the original seed;
- conservation of important interface positions;
- hotspot-retention analysis;
- physicochemical filtering;
- calculation of composite pre-docking scores;
- prioritization of candidates for downstream structural evaluation;
- export of ranked candidate tables.

The output is a reduced and ranked set of peptide candidates intended for subsequent structure-based evaluation using tools such as FoldX and Rosetta FlexPepDock.

A favourable pre-docking score does not demonstrate binding. It only identifies candidates that satisfy the selected computational criteria better than others within the generated set.

---

## Main software dependencies

The notebooks use scientific Python and structural-bioinformatics libraries, including:

- Python 3;
- Jupyter Notebook;
- NumPy;
- pandas;
- Matplotlib;
- Biopython;
- NetworkX;
- scikit-learn;
- pathlib;
- regular-expression utilities.

### FoldX

FoldX is used for computational alanine scanning and mutation-energy estimation.

FoldX must be installed separately and its executable path may need to be configured manually according to the local operating system.

Depending on the execution environment, FoldX may be:

- executed directly from a notebook;
- invoked through generated commands or scripts;
- run manually outside Jupyter;
- parsed after completion.

---

## Expected outputs

Depending on the local execution environment, Pipeline 1 may generate:

- inter-chain contact tables;
- interface-residue lists;
- chain-pair comparison tables;
- interface contact matrices;
- residue-level interface graphs;
- graph-centrality statistics;
- FoldX mutation-energy results;
- energetic-hotspot tables;
- ranked contiguous peptide windows;
- the selected seed peptide;
- generated local peptide variants;
- mutation annotations;
- pre-docking validation tables;
- ranked peptide candidates;
- diagnostic plots;
- CSV reports.

Most intermediate and final results are written to an `outputs/` directory created by the notebooks.

Large generated files, temporary FoldX runs and external software installations should normally not be committed to the repository.

---

## Scientific interpretation

This directory documents the initial PEARL computational prototype.

The workflow provides a reproducible strategy for prioritizing:

- candidate protein–protein interfaces;
- structurally central residues;
- energetic hotspots;
- contiguous interface-derived peptide windows;
- local peptide variants;
- candidates for later structure-based evaluation.

The following limitations should be considered:

- contact count alone does not establish biological relevance;
- biological-interface validation remains dependent on available structural annotations;
- graph centrality is a structural prioritization criterion, not direct experimental evidence;
- FoldX energies are approximate computational estimates;
- alanine-scanning predictions are not experimental measurements;
- a selected seed peptide is a design hypothesis;
- generated peptide variants are not confirmed binders;
- pre-docking scores do not demonstrate binding affinity;
- peptide stability, solubility, selectivity and cellular activity are not established;
- additional docking, molecular-dynamics and experimental validation are required.

The final candidates should therefore be interpreted as ranked computational hypotheses rather than confirmed EGFR inhibitors.

---

## Relationship with Pipeline 2

The second PEARL development stage is stored in:

`pipeline_2_structural_validation_and_docking/`

Pipeline 2 extends selected results from this initial prototype through two complementary directions:

1. short hotspot-centred peptide design and structural refinement;
2. CLEAR-inspired local peptide optimization followed by FoldX and Rosetta FlexPepDock validation.

The notebook numbering in this directory applies only to Pipeline 1.

Notebook prefixes used in Pipeline 2 should not be interpreted as a direct continuation of the sequence reported here. The two directories represent distinct stages of methodological development.

Pipeline 1 should remain independently understandable and reproducible as the initial PEARL prototype.

---

## Directory structure

    pipeline_1_initial_prototype/
    ├── README.md
    ├── 01_EGFR_PDB_to_Interface_Graph.ipynb
    ├── 01B_Biological_Interface_Validation.ipynb
    ├── 02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb
    ├── 03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb
    └── 04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb

---

## Project status

- **Stage:** initial PEARL computational prototype
- **Scope:** interface identification, hotspot analysis, seed extraction, variant generation and pre-docking prioritization
- **Validation level:** computational and non-experimental
- **Purpose:** methodological development, research and academic presentation
