
## Important Note : Large restart files: the serialized OpenMM systems, final 1 ns states and solvated structures required for trajectory continuation are distributed separately because of their size.
## Purpose

# PEARL – Molecular Dynamics Continuation Package

## Overview

This package contains the files required to continue the explicit-solvent molecular dynamics (MD) simulations of three selected PEARL candidates—`F0010`, `CF06`, and `CF02`—from 1 ns to 10 ns total simulation time using OpenMM.

Each continuation starts from the serialized OpenMM system and final state produced by the corresponding initial 1 ns simulation. It adds a further 9 ns without repeating energy minimization, NVT equilibration, or NPT equilibration.

The package is intended to be distributed as a ZIP archive. This `README.md` file must remain inside the `PEARL_MD_10ns_continuation/` directory before that directory is compressed.

## Selected Candidates

- **F0010** — selected PEARL candidate.
- **CF06** — selected PEARL candidate.
- **CF02** — selected PEARL candidate.

The three systems are stored in separate directories so that each continuation can be run and analyzed independently.

## Package Structure

```text
PEARL_MD_10ns_continuation/
├── README.md
├── 07d_OpenMM_continue_from_1ns_to_10ns.ipynb
├── F0010/
│   ├── states/
│   │   ├── F0010_system.xml
│   │   └── F0010_production_1ns_final_state.xml
│   └── structures/
│       └── F0010_BD_solvated.pdb
├── CF06/
│   ├── states/
│   │   ├── CF06_system.xml
│   │   └── CF06_production_1ns_final_state.xml
│   └── structures/
│       └── CF06_BD_solvated.pdb
└── CF02/
    ├── states/
    │   ├── CF02_system.xml
    │   └── CF02_production_1ns_final_state.xml
    └── structures/
        └── CF02_BD_solvated.pdb
```

For each candidate:

- `states/*_system.xml` contains the serialized OpenMM system.
- `states/*_production_1ns_final_state.xml` contains the final restart state from the initial 1 ns production run.
- `structures/*_BD_solvated.pdb` contains the corresponding solvated structure and topology reference.

## Initial Simulation Protocol

The restart files were generated using the following simulation setup:

| Parameter | Value |
|---|---|
| MD engine | OpenMM |
| Protein force field | AMBER ff14SB |
| Water model | TIP3P |
| Solvation | Explicit solvent |
| Ionic concentration | 0.15 M |
| Solvent padding | 1.0 nm |
| Nonbonded method | Particle Mesh Ewald (PME) |
| Nonbonded cutoff | 1.0 nm |
| Bond constraints | HBonds |
| Temperature | 300 K |
| Pressure | 1 bar |
| Integrator | LangevinMiddleIntegrator |
| Friction coefficient | 1 ps⁻¹ |
| Time step | 2 fs |
| Pressure coupling | MonteCarloBarostat |
| NVT equilibration | 100 ps |
| NPT equilibration | 100 ps |
| Initial production | 1 ns |
| Trajectory frame interval | 2 ps |
| Random seed base | 20260808 |

## Continuation Protocol

The notebook `07d_OpenMM_continue_from_1ns_to_10ns.ipynb` restores each candidate from its serialized system and final 1 ns state, then performs an additional 9 ns of production MD. Minimization and equilibration are not repeated because the continuation begins from the completed production state.

With a 2 fs time step:

```text
9 ns = 9,000,000 fs
9,000,000 fs / 2 fs per step = 4,500,000 steps
```

Therefore, the continuation consists of **4.5 million MD steps per system**, producing a total simulated time of **10 ns per candidate** when combined with the initial 1 ns run.

## Distribution Notes

The XML restart files can be large. They may therefore be distributed separately from the GitHub repository when repository size limits or practical download considerations make this preferable. The notebook and documentation can remain on GitHub, while the complete restart package can be shared as a separate ZIP archive.

Before creating the archive, confirm that `README.md` is located at the top level of `PEARL_MD_10ns_continuation/`, alongside the notebook. Then compress the entire directory so that the ZIP preserves the structure shown above.

