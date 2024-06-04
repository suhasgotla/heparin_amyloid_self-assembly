# Self-assembly of amyloid peptides with heparin

This directory contains source files, and protocols for the self-assembly of peptides with the glycosaminoglycan, heparin.

## Contents


```
heparin_amyloid_self-assembly
│   README.md
│
└───ff/
│   └───heparin_itps/   # topology and position restraint files 
|   |                   # for heparin molecules
|   |                   # at different degrees of polymerization (dp)
|   |                   # and rigid analogs of dp18 heparins
│   │   
│   └───protein_itps/   # topology and position restraint files 
|   |                   # for amyloid_beta 16-22 peptides
│   │   
|   └───other_itps/ 
|   |   forcefield.itp  # nonbonded interaction parameters 
|   |   water.itp       # topology files of MARTINI polarizable water
|   |   water_em.itp    # topology files of MARTINI polarizable water
|   |   ions.itp        # topology files for MARTINI monovalent ions
|   |
|   └───prompt_tables/  # tabulated potentials 
|                       # for amino-acid specific angles for ProMPT proteins
│   
└───initial_molecule_structures/    
|   |   # Coordinates of initial configurations 
|   |   # of heparin molecules, peptides, 
|   |   # and MARTINI polarizable water 
│   
└───simulation_mdps/
|   |   # MDP options for 
|   |   # energy minimization,
|   |   # NPT equilibration and production MD.
|
└───builf_16pep_1hep/   # A directory to 
    |                   # where one may perform
    |                   # the tutorial for setting up
    |                   # a sample simulation
    |
    | system.top        # A sample top file
    
```

## Tutorial: Setting up a system with 16 peptides, and 1 molecule of dp18 heparin

### Initialize coordinates of solute and solvent particles

1. Insert 16 peptides at random locations in a 10 x 10 x 10 box.
    
    ```
    > gmx insert-molecules -ci initial_molecule_structures/1pep.gro -nmol 16 -box 10 10 10 -radius .5 -o build_16pep_1hep/16pep.gro
    ```

2. Insert a dp18 heparin chain in the box
    
    ```
    > gmx insert-molecules -ci initial_molecule_structures/heparin_dp18.gro -nmol 1 -f build_16pep_1hep/16pep.gro -radius .5 -o build_16pep_1hep/16pep_1hep.gro
    ```

3. Solvate with polarizable water
    
    ```
    > gmx solvate -cp 1build_16pep_1hep/6pep_1hep.gro -cs initial_molecule_structures/water_box.gro -o build_16pep_1hep/16pep_1hep_solv.gro
    ```

4. Balance net-charge with monovalent counterions
    ```
    > gmx grompp -f protocols/em.mdp -c build_16pep_1hep/16pep_1hep_solv.gro -r build_16pep_1hep/16pep_1hep_solv.gro -p build_16pep_1hep/system.top -o build_16pep_1hep/ions.tpr
    > gmx genion -s build_16pep_1hep/ions.tpr -o build_16pep_1hep/16pep_1hep_ions.gro -neutral -p build_16pep_1hep/system.top
    ```

### Energy minimization
    ```
    > cd build_16pep_1hep
    > mkdir em
    > gmx grompp -f ../protocols/em.mdp -p system.top -o em/em -c 16pep_1hep_ions.gro
    > cd em/
    > gmx mdrun -deffnm em -v -s em.tpr -o em -tableb ../../ff/prompt_tables/*
    > cd ../
    ```

### Make directories for equilibration and production MD for one independent trial, trial_A
    ```
    > mkdir build_16pep_1hep/trial_A; mkdir build_16pep_1hep/tiral_A/eq build_16pep_1hep/trial_A/md
    ```

### NPT Equilibration: one independent replica, trial_A

    ```
    > cd build_16pep_1hep
    > gmx grompp -f ../protocols/eq.mdp -p system.top -c em/em.gro -r em/em -o trial_A/eq/eq.tpr -maxwarn 1
    > cd tiral_A/eq
    > gmx mdrun -deffnm eq -v -s eq.tpr -o eq -tableb ../../../ff/prompt_tables/*
    ```

### Production MD: one independent replica, trial_A

    ```
    > cd build_16pep_1hep
    > gmx grompp -f ../protocols/md.mdp -p system.top -c eq/eq.gro -t eq/eq.cpt -o trial_A/md/md.tpr
    > cd tiral_A/md
    > gmx mdrun -deffnm md -v -s md.tpr -o md -tableb ../../../ff/prompt_tables/*
    ```
