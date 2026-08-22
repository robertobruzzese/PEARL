# PEARL – Molecular Dynamics Continuation Package

## Important Note : Large restart files: the serialized OpenMM systems, final 1 ns states and solvated structures required for trajectory continuation are distributed separately because of their size.
## Purpose

This package contains the OpenMM systems and final simulation states required to reproduce and, in particular, **continue the molecular dynamics simulations performed in Pipeline 3 of the PEARL project**.

Three peptide–EGFR complexes are included:

- **F0010** – reference peptide
- **CF06** – counterfactual candidate showing the largest number of persistent contacts in the 1 ns MD comparison
- **CF02** – counterfactual candidate showing the highest conformational stability in the 1 ns MD comparison

The systems have already undergone preparation, solvation, minimization, equilibration and **1 ns of production molecular dynamics**.

The supplied continuation notebook is designed to resume each simulation from its final 1 ns state and perform **an additional 9 ns of production MD**, reaching a total simulated time of **10 ns**.

## Directory structure

```text
PEARL_MD_10ns_continuation/
├── README.md
├── 07d_OpenMM_continue_from_1ns_to_10ns.ipynb
├── F0010/
│   ├── F0010_system.xml
│   ├── F0010_production_1ns_final_state.xml
│   └── F0010_BD_solvated.pdb
├── CF06/
│   ├── CF06_system.xml
│   ├── CF06_production_1ns_final_state.xml
│   └── CF06_BD_solvated.pdb
└── CF02/
    ├── CF02_system.xml
    ├── CF02_production_1ns_final_state.xml
    └── CF02_BD_solvated.pdb
```

## Contents of each system

### `*_system.xml`

Serialized OpenMM `System` generated during the original Pipeline 3 simulation. It contains the force-field-derived molecular system and forces used for the MD calculation.

### `*_production_1ns_final_state.xml`

Serialized OpenMM state corresponding to the end of the original **1 ns production trajectory**.

It provides the state from which the continuation simulation is initialized.

### `*_BD_solvated.pdb`

Solvated peptide–EGFR system corresponding to the OpenMM system topology.

## Original MD protocol

The original Pipeline 3 molecular dynamics protocol used:

- **MD engine:** OpenMM
- **Protein force field:** AMBER ff14SB
- **Water model:** TIP3P
- **Explicit solvent**
- **Ionic strength:** 0.15 M
- **Solvent padding:** 1.0 nm
- **Electrostatics:** PME
- **Nonbonded cutoff:** 1.0 nm
- **Bond constraints:** HBonds
- **Temperature:** 300 K
- **Pressure:** 1 bar
- **Integrator:** LangevinMiddleIntegrator
- **Friction coefficient:** 1 ps⁻¹
- **Integration timestep:** 2 fs
- **Pressure control:** MonteCarloBarostat
- **NVT equilibration:** 100 ps
- **NPT equilibration:** 100 ps
- **Original production MD:** 1 ns
- **Trajectory frame interval:** 2 ps
- **Original random seed base:** 20260808

## Continuation protocol

The notebook:

`07d_OpenMM_continue_from_1ns_to_10ns.ipynb`

does **not** repeat system preparation, minimization or equilibration.

For each candidate it:

1. loads the serialized OpenMM system;
2. loads the corresponding solvated topology;
3. restores the final state of the 1 ns production simulation;
4. initializes an OpenMM simulation context;
5. continues production MD for an additional **9 ns**;
6. writes the continuation trajectory and thermodynamic log;
7. serializes the resulting final state corresponding to approximately **10 ns total simulation time**.

Therefore, this should be regarded as a **continuation of the existing production MD**, rather than a new independent 10 ns simulation initiated from the original structure.

## Candidates

### F0010

Reference peptide used as the control/reference system in the comparative molecular dynamics analysis.

### CF06

Counterfactual candidate that retained the largest persistent interaction network during the original 1 ns MD comparison.

### CF02

Counterfactual candidate displaying the lowest RMSD and RMSF values in the original 1 ns MD comparison, indicating greater conformational stability over the investigated interval.

## Computational considerations

Nine additional nanoseconds correspond to approximately:

**4,500,000 integration steps per system**

with a 2 fs timestep.

The total computational cost therefore depends strongly on the available OpenMM platform and hardware.

The notebook allows the computational platform to be selected according to the available machine.

## Relationship with PEARL Pipeline 3

These simulations extend the molecular-dynamics validation performed in:

**Pipeline 3 – Molecular Dynamics and Binding Energy**

of the PEARL project.

The original 1 ns simulations were used to compare F0010, CF06 and CF02 in terms of:

- RMSD;
- RMSF;
- persistence of peptide–receptor contacts;
- thermodynamic quality-control quantities.

Extending the simulations to 10 ns can provide a longer observation window for assessing whether the dynamic behaviour observed during the initial 1 ns persists over a longer timescale.

---

**PEARL project – Pipeline 3**  
Molecular Dynamics and Binding Energy
