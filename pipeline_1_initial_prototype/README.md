# Pipeline 1 – Initial Prototype

This directory contains the notebooks of the first PEARL prototype.
Ho verificato la cartella: contiene cinque notebook, contando 01B come passaggio autonomo di validazione tra il notebook 01 e il notebook 02b. L’ordine corretto da dichiarare è quindi 01 → 01B → 02b → 03b → 04b.  

Sostituirei l’attuale README molto breve con questo:

PEARL Pipeline 1 — Initial Prototype

This directory contains the first computational prototype developed for the PEARL project.

The purpose of this pipeline is to start from an EGFR crystallographic structure, identify a candidate protein–protein interface, estimate structurally and energetically important interface residues, extract a contiguous seed peptide, generate sequence variants, and prioritize peptide candidates before structural docking.

Pipeline overview

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

System studied

* Target: Epidermal Growth Factor Receptor, EGFR
* PDB structure: 3NJP
* Initial interface selected: chains B–D
* Initial contact cutoff: 4.5 Å
* Main output: a ranked collection of peptide candidates derived from an interface seed sequence

The selection of chains B–D was initially based on structural contact analysis and was subsequently examined in the biological-interface validation notebook.

Execution order

The notebooks should be executed in the following order:

1. 01_EGFR_PDB_to_Interface_Graph.ipynb

Downloads and parses the EGFR structure, identifies the available chains, calculates inter-chain contacts, and constructs a residue-level interface graph.

Main operations include:

* PDB retrieval and parsing;
* chain and residue inspection;
* heavy-atom contact detection;
* comparison of chain-pair interfaces;
* selection of a candidate interface;
* construction of an interface contact network;
* calculation of graph-based residue centrality measures.

2. 01B_Biological_Interface_Validation.ipynb

Provides an additional validation step for the interface selected in Notebook 01.

Its purpose is to determine whether the interface selected from the number of structural contacts is also compatible with the biological assembly and the known structural organization of the EGFR complex.

This notebook should be interpreted as a validation and interpretation step, rather than as an independent peptide-design stage.

3. 02b_Interface_Hotspot_AlanineScanning_FoldX_BuildModel_FIXED.ipynb

Evaluates the energetic contribution of interface residues through computational alanine scanning using FoldX.

Main operations include:

* preparation of the selected interface structure;
* mutation of interface residues to alanine;
* FoldX BuildModel execution;
* estimation of mutation-induced energy changes;
* identification of candidate energetic hotspots;
* integration of energetic, contact, and graph-based information.

Residues with a sufficiently positive mutation-induced energy change are considered candidate hotspots because their replacement with alanine is predicted to destabilize the interface.

4. 03b_Contiguous_Peptide_Window_Diffusion_EnergeticHotspots_FIXED.ipynb

Searches the selected interface chain for contiguous peptide windows that retain important interface residues and energetic hotspots.

Main operations include:

* construction and ranking of contiguous sequence windows;
* selection of a seed peptide;
* representation of interface and hotspot positions within the seed;
* generation of conservative local sequence variants;
* candidate scoring based on sequence, structural, and energetic constraints.

The initial prototype selected a 30-residue seed peptide from chain D:

YIEALDKYACNCVVGYIGERCQYRDLKWWE

The generated candidates represent local variants of this seed and are not yet validated binders.

5. 04b_Candidate_Validation_PreDocking_EnergeticHotspots.ipynb

Performs pre-docking validation and ranking of the generated peptide candidates.

Main operations include:

* sequence-validity checks;
* conservation of important interface positions;
* hotspot-retention analysis;
* physicochemical filtering;
* comparison with the original seed;
* calculation of a composite pre-docking score;
* prioritization of candidates for subsequent structural analysis.

The output of this notebook is a reduced and ranked set of candidates intended for later FoldX evaluation, peptide docking, and molecular-dynamics analysis.

Main software dependencies

The notebooks use Python scientific and structural-bioinformatics libraries, including:

* Python 3;
* Jupyter Notebook;
* NumPy;
* pandas;
* Matplotlib;
* Biopython;
* NetworkX;
* scikit-learn;
* FoldX.

FoldX is external software and must be installed separately. Its executable path may need to be configured manually according to the local operating system.

Expected outputs

Depending on the execution environment, the pipeline generates files such as:

* interface-contact tables;
* interface-residue lists;
* graph-centrality statistics;
* FoldX mutation-energy results;
* energetic-hotspot tables;
* ranked contiguous peptide windows;
* the selected seed peptide;
* generated peptide variants;
* pre-docking candidate rankings;
* CSV reports and diagnostic plots.

Most intermediate results are written to an outputs/ directory created by the notebooks.

Scientific interpretation

This directory documents an initial PEARL prototype.

The workflow provides a computational strategy for prioritizing residues, peptide windows, and candidate sequences. However:

* contact count alone does not establish biological relevance;
* FoldX energies are computational estimates;
* graph centrality is a structural prioritization criterion, not direct experimental evidence;
* generated peptide candidates are hypotheses;
* pre-docking scores do not demonstrate binding;
* the candidates require further structural and experimental validation.

Subsequent PEARL pipelines may revise the interface selection, candidate-generation procedure, FoldX evaluation, docking protocol, or final ranking.

Relationship with later pipelines

The notebook numbering in this directory applies only to Pipeline 1.

Later notebooks stored in other pipeline directories represent subsequent stages or revised implementations and should not be interpreted as a direct continuation of the numbering used here.

In particular, this initial prototype should remain reproducible and distinguishable from later workflows involving revised dataset construction, structural preparation, FoldX complex evaluation, Rosetta FlexPepDock refinement, or other post-meeting developments.

Project status

Status: initial research prototype
Purpose: methodological development and preliminary candidate generation
Validation level: computational, non-experimental
Intended use: research and educational purposes

Questo README chiarisce anche un punto importante: 01B non è semplicemente una variante del notebook 01, ma un passaggio di controllo della scelta biologica dell’interfaccia. In GitHub devi aprire README.md, cliccare sulla matita, sostituire il testo attuale e fare Commit changes.
