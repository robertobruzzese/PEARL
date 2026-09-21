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
    structural and biological-context interface assessment
        ↓
    FoldX BuildModel mutation-energy sensitivity analysis
        ↓
    FoldX AnalyseComplex interaction-energy sensitivity assessment
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
- **Chain identity:** chain `B` = extracellular EGFR; chain `D` = EGF
- **Initial heavy-atom contact cutoff:** 4.5 Å
- **Reference peptide source:** a contiguous region extracted from the selected protein–protein interface
- **Main output:** a ranked collection of peptide candidates derived from an interface seed sequence

The `B–D` interface was initially selected by comparing the number of inter-chain structural contacts. This choice was subsequently examined through structural and biological-context assessment. The analysis does not by itself experimentally validate the interface.

The analyses contained in this directory are computational and do not constitute experimental evidence of peptide binding, inhibition or biological activity.

---

## Notebooks included

- `01_EGFR_PDB_to_Interface_Graph.ipynb`
- `01B_Biological_Interface_Validation.ipynb`
- `02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb`
- `02c_FoldX_Binding_Hotspot_Validation.ipynb`
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
    02c_FoldX_Binding_Hotspot_Validation
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

This notebook provides an additional structural and biological-context assessment of the interface selected in Notebook `01`.

Its purpose is to determine whether the interface selected from structural contact analysis is also compatible with:

- the biological assembly of the crystallographic structure;
- the known organization of the EGFR complex;
- chain identity and symmetry;
- structural annotations associated with the PDB entry;
- the biological plausibility of the selected chain pair.

This notebook should be interpreted as a structural/contextual assessment and interpretation stage rather than as experimental biological validation or an independent peptide-design procedure.

Its role is to reduce the risk of carrying an interface forward solely because it contains the largest number of geometric contacts.

---

## `02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb`

This notebook performs FoldX `BuildModel` alanine-mutation calculations and uses the resulting mutation-energy changes as an operational sensitivity measure for peptide design.

Main operations include:

- preparation of the selected B–D interface structure;
- definition of chain-D residues to be scanned;
- mutation of selected residues to alanine with FoldX `BuildModel`;
- parsing of mutation-induced total-energy changes;
- application of the historical operational threshold `> 1.5 kcal/mol`;
- integration of BuildModel sensitivity, contact and graph-based information;
- construction of ranked interface-residue and contiguous-window tables.

The historical `> 1.5 kcal/mol` classification is retained as a **BuildModel-derived energetic design criterion**. It must not be interpreted as a direct binding ΔΔG measurement, an experimental hotspot classification, or proof that a residue contributes a specific amount to EGF–EGFR binding affinity.

In the historical 02b ranking, the first 30-residue window was D:21–50 and D:22–51 was second.

---

## `02c_FoldX_Binding_Hotspot_Validation.ipynb`

This notebook adds an interaction-specific FoldX assessment of the extracellular EGF–EGFR B–D interface using `AnalyseComplex`. Chain `B` is extracellular EGFR and chain `D` is EGF.

The repaired wild-type B–D complex has a FoldX interaction energy of `-36.5948`. For each selected chain-D residue, alanine-mutant complexes generated with `BuildModel` are evaluated with `AnalyseComplex`, and the mutant interaction energy is compared with the wild-type value.

The resulting quantity is described as **FoldX interaction-energy sensitivity to alanine mutation**. It is a computational sensitivity measure and not an experimentally measured binding free-energy change.

The associated A07 analysis reassessed the historical contiguous-window selection. For the two leading 30-residue windows, cumulative signed interaction-energy sensitivity was:

- D:22–51: `22.66614`;
- D:21–50: `22.51558`.

The difference is `+0.15056`, corresponding to the replacement of M21 by E51 in the two windows. Both windows contained 21 tested residues. D:22–51 was therefore adopted as the updated downstream seed. The difference is small, and D:22–51 should not be interpreted as optimal under every possible ranking or normalization criterion.

---

## `03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb`

This notebook uses the updated A07 seed selection and generates local peptide variants while retaining important interface positions and BuildModel-derived energetic design anchors.

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

The historical 02b ranking placed D:21–50 first. After the interaction-specific 02c/A07 reassessment, the updated operational seed is the 30-residue chain-D window D:22–51:

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
- retention analysis for BuildModel-derived energetic design anchors;
- physicochemical filtering;
- calculation of composite pre-docking scores;
- prioritization of candidates for downstream structural evaluation;
- export of ranked candidate tables.

The updated output contains the D:22–51 seed plus 10 selected non-seed candidates intended for subsequent structure-based evaluation. The principal downstream files are `outputs/top_diffusion_energetic_peptides_summary.csv` and `outputs/top_diffusion_energetic_peptides_pre_docking.fasta`.

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

FoldX is used both for `BuildModel` mutation-energy sensitivity calculations and for the interaction-specific `AnalyseComplex` assessment.

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
- BuildModel mutation-energy sensitivity and design-anchor tables;
- AnalyseComplex interaction-energy sensitivity results;
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
- BuildModel-derived energetic design anchors and interaction-energy-sensitive residues;
- contiguous interface-derived peptide windows;
- local peptide variants;
- candidates for later structure-based evaluation.

The following limitations should be considered:

- contact count alone does not establish biological relevance;
- structural and biological-context assessment remains dependent on available structural annotations and does not constitute experimental interface validation;
- graph centrality is a structural prioritization criterion, not direct experimental evidence;
- FoldX energies are approximate computational estimates;
- BuildModel mutation-energy changes are not direct binding ΔΔG values;
- AnalyseComplex interaction-energy sensitivities are computational and not experimental measurements;
- a selected seed peptide is a design hypothesis;
- generated peptide variants are not confirmed binders;
- pre-docking scores do not demonstrate binding affinity;
- peptide stability, solubility, selectivity and cellular activity are not established;
- additional docking, molecular-dynamics and experimental validation are required.

The final candidates should therefore be interpreted as ranked computational peptide hypotheses for subsequent structure-based evaluation. The workflow does not establish EGF competition, EGFR inhibition, binding affinity, biological response or therapeutic efficacy.

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
    ├── 02c_FoldX_Binding_Hotspot_Validation.ipynb
    ├── 03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb
    └── 04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb

---

## Project status

- **Stage:** initial PEARL computational prototype
- **Scope:** extracellular EGF–EGFR interface identification, structural/contextual assessment, FoldX mutation- and interaction-energy sensitivity analysis, seed extraction, variant generation and pre-docking prioritization
- **Validation level:** computational and non-experimental
- **Purpose:** methodological development, research and academic presentation
