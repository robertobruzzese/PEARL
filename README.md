![PEARL --- Peptide Extraction and AI-guided Refinement for Ligand
design](pearl.png) \# PEARL --- Peptide Extraction and AI-guided
Refinement for Ligand Design

**From Dimers to Drugs**

PEARL is a computational drug-discovery research prototype that
reverse-engineers the extracellular EGF--EGFR protein--protein interface
and uses structural bioinformatics, molecular modelling, molecular
dynamics (MD), protein AI and cheminformatics to prioritize peptide and
small-molecule binding and modulation hypotheses.

The current implementation is developed around the **extracellular
EGF--EGFR complex (PDB `3NJP`)**, focusing on the interface between
**EGFR chain B and EGF chain D**.

> **Repository status:** five implemented computational pipelines
> spanning interface analysis, peptide design, molecular dynamics,
> AI-guided peptide redesign, MD-derived pharmacophore construction and
> small-molecule lead prioritization. All reported leads remain
> computational hypotheses; no experimental binding, inhibition,
> efficacy or safety validation is claimed.

------------------------------------------------------------------------

> **Interpretation of computational evidence.** PEARL prioritizes
> candidates on the basis of computational evidence for structural
> compatibility, interaction, dynamics, docking and energetic
> descriptors. These results support **binding/modulation hypotheses**;
> they do not by themselves demonstrate competition with EGF, biological
> inhibition of EGFR, functional response, or therapeutic efficacy. Any
> inhibitory mechanism therefore remains a hypothesis to be tested
> experimentally.

## Workflow at a glance

``` text
Extracellular EGF–EGFR complex (3NJP: EGFR chain B–EGF chain D)
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

PEARL follows a layered-validation philosophy: no surrogate score,
force-field energy, docking output, structure prediction or short MD
trajectory is treated as sufficient evidence on its own. Cross-method
agreement is used for **prioritization**, not as proof of biological
activity.

------------------------------------------------------------------------

## Biological system

  Item                      Current implementation
  ------------------------- -----------------------------------------
  Target                    Epidermal Growth Factor Receptor (EGFR)
  Reference structure       PDB `3NJP`
  Studied interface         Chains `B–D`
  Receptor chain            `B` (extracellular EGFR)
  Native ligand chain       `D` (EGF)
  Short natural reference   `F0010`
  F0010 sequence            `IGERCQYRDLK`
  F0010 mapping             Chain D residues 38--48

F0010 is a contiguous, hotspot-rich fragment derived from the native
interface and serves as the principal short-peptide reference throughout
the repository.

------------------------------------------------------------------------

## Repository structure

``` text
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
│                 BoTorch proposal and multi-objective integration
└── pipeline_5_pharmacophore_and_small_molecule_leads/
    ├── README.md
    └── 09a–09k  MD pharmacophore, REINVENT4, chemistry filtering,
                  DiffDock, Vina, historical lead selection,
                  lead MD/endpoint analysis and CUDA replicas
```

Some computationally expensive notebooks have test and production
variants. The pipeline-level README files document their execution
order, inputs, parameters and output contracts.

------------------------------------------------------------------------

## Implementation status

  ------------------------------------------------------------------------
  Pipeline                Status                  Implemented scope
  ----------------------- ----------------------- ------------------------
  **1 --- Initial         Complete (historical    Interface
  prototype**             branch)                 identification, graph
                                                  construction,
                                                  biological-interface
                                                  checks, FoldX alanine
                                                  scanning, hotspots and
                                                  initial peptide windows

  **2 --- Peptide design  Complete                Hotspot-centered
  and structural                                  miniaturization, F0010
  validation**                                    selection, 539-variant
                                                  local landscape, GNN
                                                  oracle, Direct CLEAR,
                                                  FoldX, FlexPepDock,
                                                  adjacent-pair scoring
                                                  and VAE-CLEAR comparison

  **3 --- MD and endpoint Complete as a           OpenMM explicit-solvent
  energetics**            comparative prototype   simulations, RMSD/RMSF,
                                                  contact persistence,
                                                  hotspot integration and
                                                  single-trajectory
                                                  MM/GBSA-like endpoint
                                                  comparisons; a 10 ns
                                                  continuation notebook is
                                                  also included

  **4 --- AI-guided       Implemented through     ESM-2, ProteinMPNN,
  peptide design**        `08j`                   FoldX/Rosetta, 1 ns MD,
                                                  harmonized endpoint
                                                  energetics, ESMFold and
                                                  developability proxies;
                                                  `08i` uses the canonical
                                                  FULL_COHORT-2 table and
                                                  executes a BoTorch
                                                  `SingleTaskGP` + LogEI
                                                  proposal over 24 designs
                                                  (5 observed, 19
                                                  unobserved); `08j`
                                                  integrates evidence
                                                  tiers and
                                                  multi-objective/Pareto
                                                  decision support

  **5 --- Pharmacophore   Implemented through     MD-derived
  and small molecules**   `09k`; audit A24--A26c  pharmacophore, REINVENT4
                          completed               generation followed by
                                                  screening, chemistry
                                                  rescue, whole-chain
                                                  DiffDock, local Vina
                                                  refinement, historical
                                                  09g prioritization,
                                                  09h/09i follow-up,
                                                  free-ligand 09j and
                                                  three 10 ns
                                                  EGFR--MOL00583 replicas
                                                  in 09k; native
                                                  EGF-interface relevance
                                                  was audited separately
  ------------------------------------------------------------------------

------------------------------------------------------------------------

## Pipeline 1 --- Interface and hotspot prototype

Pipeline 1 established the original PEARL logic:

``` text
3NJP → B–D interface → residue-contact graph → FoldX alanine scan
     → energetic hotspots → contiguous peptide windows
     → initial candidate ranking
```

This branch is retained as the historical prototype and provenance for
later structural and energetic choices.

------------------------------------------------------------------------

## Pipeline 2 --- Peptide design, CLEAR and structural validation

Pipeline 2 contains two complementary peptide-design branches.

### Hotspot-centered miniaturization

Natural interface fragments are ranked structurally and energetically,
prepared for guided docking and refined with Rosetta FlexPepDock. The
11-residue peptide **F0010 (`IGERCQYRDLK`)** emerged as the primary
miniaturized reference.

### Local CLEAR and VAE optimization

A constrained **539-variant** sequence landscape around F0010 supports a
differentiable GNN-based local oracle and Direct CLEAR-inspired
counterfactual search. FoldX and FlexPepDock are then used as explicit
structural filters. A true VAE extension explores the same local
landscape in latent space.

Direct CLEAR found 36 successful sequences and VAE-CLEAR found four; all
four VAE successes were also recovered by Direct CLEAR. This is useful
algorithmic convergence, but not independent biological validation
because the methods share the underlying local dataset and oracle.

Important recurring mutation patterns include `D9E` and `C5S + D9E`.
Rosetta results indicate that the effect of `D9E` depends strongly on
the accompanying mutation.

------------------------------------------------------------------------

## Pipeline 3 --- Molecular dynamics and comparative endpoint energetics

Pipeline 3 moves from static models to explicit-solvent MD using OpenMM
with AMBER ff14SB, TIP3P water, approximately 0.15 M NaCl, 300 K and 1
bar. The primary production comparison uses 1 ns trajectories; these are
short comparative simulations, not evidence of long-timescale
convergence.

### Interface-level results

-   147 unique B--D contacts were observed during the prototype MD.
-   80 contacts persisted in at least 50% of analyzed frames.
-   All 11 F0010 positions were supported by MD-persistent interface
    contacts.
-   Six of the 11 positions were supported by both FoldX hotspot
    analysis and MD persistence.

### Peptide comparison

  -------------------------------------------------------------------------------------------
  Candidate   Sequence          Mean RMSD   Mean RMSF   Persistent          Mean     Endpoint
                                      (Å)         (Å)     contacts   persistence        proxy
                                                              ≥50%                 (kcal/mol)
  ----------- --------------- ----------- ----------- ------------ ------------- ------------
  `CF02`      `IGERSQYRELK`     **0.901**   **0.737**           43     **0.603**     −56.62 ±
                                                                                         4.63

  `CF06`      `IGERCQYRELR`         1.293       0.904       **45**         0.529   **−65.66 ±
                                                                                       6.80**

  `F0010`     `IGERCQYRDLK`         1.966       1.164           43         0.541     −59.46 ±
                                                                                         5.96
  -------------------------------------------------------------------------------------------

`CF02` is the strongest short-timescale dynamic-stability reference,
whereas `CF06` is the strongest structural/endpoint-energy reference.
The endpoint calculation is a comparative, single-trajectory
**MM/GBSA-like proxy**; it is not a rigorous absolute binding free
energy.

------------------------------------------------------------------------

## Pipeline 4 --- Protein AI and peptide redesign (`08a–08j`)

Pipeline 4 adds sequence- and structure-conditioned AI while preserving
downstream physical and structural checks.

  ----------------------------------------------------------------------------------------
  Notebook                Implemented role                 Scientific status
  ----------------------- -------------------------------- -------------------------------
  `08a`                   ESM-2 masked                     Sequence-plausibility prior;
                          pseudo-log-likelihood,           not binding evidence
                          pseudo-perplexity and embeddings 

  `08b`                   Constrained ProteinMPNN redesign 100 samples, 24 unique designs;
                          of peptide chain D               not affinity prediction

  `08c`                   Integrated ESM-2/ProteinMPNN     Transparent heuristic
                          shortlist                        prioritization

  `08d`                   FoldX and Rosetta FlexPepDock    Static structural/energetic
                          validation                       filtering

  `08e`                   Explicit-solvent MD for top      1 ns comparative prototype
                          AI-derived peptides              

  `08f`                   Harmonized endpoint-energy       Comparative proxy; not absolute
                          comparison                       ΔG

  `08g`                   ESMFold sequence/file QC, pLDDT  Complementary structural
                          and geometry comparison          evidence; not independent
                                                           affinity evidence

  `08h`                   Physicochemical/developability   Canonical FULL_COHORT-2 table;
                          screening                        sequence-derived proxies used
                                                           where authentic CamSol is
                                                           unavailable

  `08i`                   Evidence integration and         Canonical FULL_COHORT-2: 24
                          Bayesian optimization            designs, 5 observed, 19
                                                           unobserved; `SingleTaskGP` +
                                                           LogEI executed; proposed
                                                           `MPNN_POOL_023 = VGARNQYRDLN`

  `08j`                   Evidence-tier and                Canonical EVIDENCE_TIERS-2
                          multi-objective integration      decision support; preserves
                                                           measured/computed/proxy
                                                           provenance
  ----------------------------------------------------------------------------------------

### AI-derived peptide results

The five-candidate ProteinMPNN shortlist was initially led by
`MPNN_NEW_01` (`TGPRNQYRDLP`). Static FoldX/Rosetta analysis favored
`MPNN_NEW_05` (`IGPRHQYRDLP`), and both advanced to MD.

  Evidence domain                 `MPNN_NEW_01`       `MPNN_NEW_05`
  ----------------------------- --------------- -------------------
  Mean peptide RMSD, 1 ns (Å)         **1.268**               1.417
  Mean contact persistence            **0.599**               0.535
  Endpoint proxy (kcal/mol)       −46.41 ± 4.66   **−53.67 ± 6.43**
  ESMFold mean pLDDT                  **72.56**               67.27
  ESMFold global Cα RMSD (Å)           **4.49**                5.07
  GRAVY proxy                         **−1.94**               −1.44

These methods describe different evidence domains and are not
interchangeable affinity measurements.

### 08i--08j audit update

The earlier repository summary described `08i` only as BoTorch
readiness. The audited canonical run is later and more complete:
**FULL_COHORT-2** contains 24 designs, of which 5 have observed
downstream evidence and 19 form the unobserved proposal pool. `08i` fits
a BoTorch `SingleTaskGP`, applies LogEI and proposes `MPNN_POOL_023`
(`VGARNQYRDLN`). This is a model-based proposal for future evaluation,
not experimental evidence and not a validated affinity improvement.

`08j` uses the canonical **EVIDENCE_TIERS-2** integration. It keeps
heterogeneous evidence and provenance explicit rather than treating
every computational output as an independent biological confirmation.
Authentic CamSol results are not claimed.

------------------------------------------------------------------------

## Pipeline 5 --- MD-derived pharmacophore and small-molecule leads (`09a–09k`)

Pipeline 5 translates persistent peptide--EGFR interactions into a
pharmacophore/small-molecule discovery funnel and then follows the
historically selected MOL00583 configuration through MD and endpoint
analyses. The audit separates **historical computational
prioritization** from **native EGF-interface relevance**.

``` text
09a  MD-derived pharmacophore extraction
 ↓
09b  screening-ready consolidation
 ↓
09c  REINVENT4 generation/acquisition + downstream pharmacophore screening
 ↓
09d  chemistry filtering / near-miss rescue
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
A26b native-interface check
 ↓
09i  GBn2/ACE endpoint characterization
 ↓
09j  free MOL00583 in water
 ↓
09k  3 × 10 ns EGFR–MOL00583 replicas
 ↓
A26c native-interface check
```

### Generation, screening and chemistry

REINVENT4 was used for unconstrained de novo sampling (5,000 SMILES
requested); 1,000 molecules were subsequently analyzed and screened
against the pharmacophore. This is **generation followed by downstream
pharmacophore screening**, not pharmacophore-conditioned reinforcement
learning.

The two strict hits (`MOL00336`, `MOL00570`) failed downstream chemistry
criteria. The eight molecules advanced to docking therefore remain
`NEAR_MISS_RESCUE`, `NEAR_MISS_MANDATORY_ONLY`, `CHEM_PASS` candidates.

### 09e / A24 --- whole-chain docking versus EGF-interface relevance

The historical 09e DiffDock ranking selected one pose per candidate from
whole-chain/full-surface docking. A24 then explicitly compared those
poses with the 43 residues of the native EGF--EGFR interface.

Among the four candidates advanced to 09f, the historical selected poses
of `MOL00583`, `MOL00600` and `MOL00273` were off-site; `MOL00484`
retained an EGF-interface-relevant selected pose. This means:

``` text
historical whole-chain 09e ranking ≠ EGF-interface-specific ranking
```

### 09f / A25 --- local Vina refinement

09f locally re-scores/refines the selected DiffDock poses; it is not an
independent global docking campaign. A25 showed that the old ligand-only
`GetBestRMS` quantity should not be called pose stability.
Receptor-frame RMSDs were approximately 1.12--1.49 Å across the four
finalists, consistent with limited local rearrangement, not proof of
binding-site stability.

### 09g / A26 --- historical prioritization

The original six 09g criteria are partially dependent computational
criteria, not six independent evidence streams. The historical ranking
is preserved as provenance:

    Historical rank Candidate    Historical status                        Top-two support
  ----------------- ------------ -------------------------------------- -----------------
                  1 `MOL00583`   Historical Tier-1 computational lead                 4/6
                  2 `MOL00484`   Historical Tier-2 finalist                           3/6
                  3 `MOL00600`   Historical Tier-2 finalist                           3/6
                  4 `MOL00273`   Historical Tier-2 finalist                           2/6

A26 does **not** automatically rerank the four molecules. `MOL00583`
remains the historical 09g lead, while `MOL00484` is the finalist that
retains site-relevant structural support and therefore merits
site-focused follow-up.

### 09h--09k / A26b--A26c --- what the downstream simulations actually validate

09h propagated the historical 09f MOL00583 pose for 1 ns. A26b
reanalyzed the existing trajectory against the true native EGF
interface: **0/400 analyzed frames** contacted any of the 43 interface
residues; the minimum ligand--interface distance was 9.167 Å.

09i then produced a mean GBn2/ACE endpoint proxy of approximately
**−20.10 ± 2.37 kcal/mol** for that simulated configuration. Because the
source trajectory is off-site, this value characterizes the off-site
EGFR--MOL00583 configuration. It is not a rigorous binding ΔG and does
not yield Kd or Ki.

09j remains a valid free-water conformational/hydration characterization
of MOL00583. Its historical `free` versus `bound` comparison must be
understood as free ligand versus the **off-site EGFR-associated 09h
configuration**, not a thermodynamic EGF-site binding comparison.

09k contains three existing 10 ns EGFR--MOL00583 replicas. A26c analyzed
800 post-discard frames per replica, **2400 frames total**. No native
EGF-interface contact was observed in any analyzed frame. Minimum
ligand--interface distances were approximately **9.04, 7.73 and 9.80 Å**
for replicas 1--3.

Therefore 09h--09k support persistence/evolution of the **off-site
configuration that was actually simulated**. They do not demonstrate
retention at, or spontaneous migration to, the native EGF--EGFR
interface. This does not exclude other possible MOL00583 binding modes.

------------------------------------------------------------------------

## Current prioritized leads

### Peptides

  ---------------------------------------------------------------------------------
  Candidate               Sequence                Current role
  ----------------------- ----------------------- ---------------------------------
  `CF06`                  `IGERCQYRELR`           Strongest integrated structural
                                                  and endpoint-energy reference

  `CF02`                  `IGERSQYRELK`           Strongest 1 ns dynamic-stability
                                                  reference; CLEAR/VAE convergence

  `MPNN_NEW_01`           `TGPRNQYRDLP`           Best equal-weight Pipeline 4
                                                  exploratory balance; stronger
                                                  MD/developability-proxy/ESMFold
                                                  profile among the two AI
                                                  finalists

  `MPNN_NEW_05`           `IGPRHQYRDLP`           Static FoldX/Rosetta and
                                                  endpoint-energy-favored AI
                                                  alternative

  `F0010`                 `IGERCQYRDLK`           Native interface-derived
                                                  reference/control
  ---------------------------------------------------------------------------------

These candidates form a **multi-method shortlist**, not a universal
affinity ranking.

### Small molecules

  -----------------------------------------------------------------------
  Candidate                           Current role after audit
  ----------------------------------- -----------------------------------
  `MOL00583`                          **Historical 09g Tier-1
                                      computational lead**; propagated
                                      09e/09f pose is off-site and
                                      09h/09k do not show native
                                      EGF-interface contact

  `MOL00484`                          Historical rank 2; retains native
                                      EGF-interface-relevant structural
                                      support and is appropriate for
                                      site-focused follow-up

  `MOL00600`                          Historical rank 3; strong Vina
                                      support, but historical selected
                                      pose is off-site

  `MOL00273`                          Historical rank 4; strongest
                                      original whole-chain DiffDock
                                      ranking, but historical selected
                                      pose is off-site
  -----------------------------------------------------------------------

These roles are **not a new EGF-site ranking**. The audit preserves
historical provenance and identifies which conclusions require
site-specific reassessment.

------------------------------------------------------------------------

## Software and external dependencies

The repository uses multiple environments because several external tools
have distinct installation and licensing requirements.

### Core scientific Python stack

-   Python and Jupyter Notebook
-   NumPy, pandas, SciPy and Matplotlib
-   scikit-learn
-   PyTorch
-   Biopython
-   NetworkX
-   RDKit

### Structural modelling and peptide design

-   FoldX
-   Rosetta, including FlexPepDock
-   ESM-2 / `fair-esm`
-   ProteinMPNN
-   ESMFold-generated structure inputs

### Molecular dynamics and trajectory analysis

-   OpenMM
-   PDBFixer
-   MDAnalysis
-   AMBER-family force fields and implicit-solvent models as specified
    by the notebooks

### Small-molecule generation and docking

-   REINVENT4
-   DiffDock and its PyTorch Geometric/e3nn dependencies
-   AutoDock Vina
-   Meeko

FoldX, Rosetta, ProteinMPNN, REINVENT4, DiffDock, Vina and model
weights/databases may require separate installation, licenses, downloads
and local path configuration. Exact versions, production/test switches
and platform-specific setup should be recorded alongside each executed
run for reproducibility.

Authentic CamSol should not be listed as a completed computational
dependency of the current results because it was not executed. **BoTorch
was executed in the audited canonical 08i FULL_COHORT-2 workflow**
(`SingleTaskGP` + LogEI) to generate a model-based proposal; that
proposal is computational and remains unevaluated downstream.

------------------------------------------------------------------------

## Interpretation and limitations

-   **No experimental validation:** the repository contains no direct
    measurements of binding, inhibition, selectivity, cellular activity,
    toxicity, pharmacokinetics or efficacy.
-   **Model-system scope:** conclusions are specific to the
    extracellular EGFR `3NJP` B--D structural context and the
    preparation choices used here.
-   **Local peptide search:** CLEAR and VAE-CLEAR operate on a
    deliberately local landscape around F0010; the oracle learns
    computational labels rather than experimental affinity.
-   **Shared evidence is not independence:** Direct CLEAR and VAE-CLEAR
    share data and an oracle; Pipeline-5 docking/refinement criteria
    also share structures and scoring ancestry. Cross-method agreement
    must not be counted mechanically as independent biological
    confirmation.
-   **Approximate static scores:** FoldX and Rosetta values are
    model-dependent and are not experimental affinities.
-   **Short peptide MD:** the principal 1 ns peptide trajectories
    support comparative prototyping but cannot establish long-timescale
    convergence.
-   **Endpoint energetics:** peptide and small-molecule endpoint values
    are comparative/model-dependent proxies, not rigorous absolute
    binding free energies, Kd or Ki.
-   **ESMFold:** confidence and RMSD for isolated short peptides do not
    establish receptor-bound conformations or affinity.
-   **Developability:** authentic CamSol was not executed in the audited
    workflow; sequence-derived physicochemical descriptors remain
    proxies.
-   **Bayesian optimization:** the audited canonical 08i executes a
    BoTorch `SingleTaskGP` + LogEI proposal over the FULL_COHORT-2 pool.
    `MPNN_POOL_023` is a model-based proposal, not a validated improved
    peptide.
-   **Pharmacophore generation:** REINVENT4 sampling was unconstrained;
    pharmacophore matching occurred downstream.
-   **Near-miss rescue:** all docked Pipeline-5 finalists remain
    chemically acceptable mandatory-only pharmacophore near misses.
-   **Whole-chain versus native-site docking:** historical 09e ranking
    was based on whole-chain/full-surface docking and must not be
    treated as an EGF-interface ranking.
-   **MOL00583 downstream simulations:** A26b found 0/400
    native-interface-contact frames in 09h; A26c found 0/2400 across the
    three 09k replicas. These simulations characterize an off-site
    EGFR--MOL00583 configuration, not validated EGF-site binding.
-   **Historical ranking versus current interpretation:** MOL00583
    remains the historical 09g lead; the audit does not automatically
    rerank the finalists. MOL00484 retains site-relevant structural
    support for follow-up.
-   **Drug-likeness is not a drug:** cheminformatic filters, docking
    scores and Pareto analyses identify follow-up hypotheses, not safe
    or effective medicines.

------------------------------------------------------------------------

## Reproducibility principles

For every production result, preserve:

1.  exact candidate identifiers and sequences/SMILES;
2.  input structures and chain/residue mappings;
3.  software and model versions;
4.  random seeds and production/test switches;
5.  executable/database/model-weight paths;
6.  raw outputs before aggregation;
7.  hashes or manifests where notebooks provide them;
8.  explicit distinction between calculated values, imported external
    outputs and proxies.

The pipeline README files and notebook QC cells are the authoritative
sources for stage-specific execution details.

------------------------------------------------------------------------

## Final repository status

PEARL now implements and audits a five-pipeline computational prototype:

1.  the extracellular EGFR B--D interface is identified and
    characterized;
2.  hotspot-rich EGF-derived peptides are extracted, miniaturized and
    locally optimized;
3.  prioritized peptides are compared by static modelling,
    explicit-solvent MD and endpoint proxies;
4.  ESM-2 and ProteinMPNN expand peptide design, followed by
    FoldX/Rosetta, MD, endpoint, ESMFold, developability-proxy analysis,
    a canonical BoTorch proposal (`MPNN_POOL_023`) and
    evidence-tier/multi-objective integration;
5.  peptide--EGFR MD ensembles are translated into a pharmacophore and
    small-molecule funnel, followed by whole-chain docking, local
    refinement, historical lead selection and explicit native-interface
    audit through A24--A26c.

The Pipeline-5 audit materially refines the final interpretation.
`MOL00583` remains the **historical computational lead selected by
09g**, but its propagated pose and the 09h/09k trajectories are off-site
relative to the native EGF--EGFR interface. `MOL00484` retains
site-relevant structural support, without being automatically promoted
to a new overall lead.

The central result is therefore not a validated inhibitor. It is a
reproducible chain of computational evidence that narrows an
extracellular EGF--EGFR design problem to peptide and small-molecule
hypotheses, while explicitly recording where historical prioritization
and native-site structural evidence agree or diverge. All candidates
remain research hypotheses requiring further site-focused computation
and experimental validation.

------------------------------------------------------------------------

## Selected CSV outputs

The [selected outputs](selected_outputs/README.md) folder contains 56
curated CSV tables from Pipelines 1--5. It includes peptide sequences,
molecule SMILES, rankings, comparison results and the later 09h--09k
analyses. Its index records each CSV's source path and any missing
files. Production results are used where test and production variants
coexist. Structure and trajectory files referenced by these tables are
not included.
