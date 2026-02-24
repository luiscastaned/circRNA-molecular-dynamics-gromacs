# circRNA-molecular-dynamics-gromacs
Complete molecular dynamics (MD) simulation and structural analysis pipeline for circular RNA (circRNA) using GROMACS and AMBER14SB-OL15 corrections.
# 🧬 circRNA Molecular Dynamics Simulation using GROMACS

---

## 📖 Overview

This repository contains the full molecular dynamics (MD) workflow for a circular RNA (circRNA) system using GROMACS.

The pipeline is organized into real simulation phases:

1️⃣ System Preparation  
2️⃣ Energy Minimization  
3️⃣ NVT Equilibration (Temperature stabilization)  
4️⃣ NPT Equilibration (Pressure & density stabilization)  
5️⃣ Production MD (Scientific simulation)  
6️⃣ Structural Analysis (RMSD, RMSF, Rg, Contacts, H-bonds)

Energy minimization has already been completed.

The MD simulation starts from:

```
em.gro
```

---

# 🚀 Molecular Dynamics Pipeline

---

# 🟢 STEP 3 — NVT Equilibration (Heating to 300K)

## Create `nvt.mdp`

```bash
cat << EOF > nvt.mdp
title                   = NVT equilibration
integrator              = md
nsteps                  = 50000
dt                      = 0.002

nstxout                 = 1000
nstvout                 = 1000
nstenergy               = 1000
nstlog                  = 1000

continuation            = no
constraint_algorithm    = lincs
constraints             = h-bonds
cutoff-scheme           = Verlet
ns_type                 = grid
coulombtype             = PME
rcoulomb                = 1.0
rvdw                    = 1.0

tcoupl                  = V-rescale
tc-grps                 = RNA Water_and_ions
tau_t                   = 0.1 0.1
ref_t                   = 300 300

pcoupl                  = no

pbc                     = xyz
EOF
```

## Run NVT

```bash
gmx grompp -f nvt.mdp -c em.gro -p topol.top -o nvt.tpr -maxwarn 1
gmx mdrun -deffnm nvt
```

⏱ This heats the system to 300K under constant volume.

---

# 🔵 STEP 4 — NPT Equilibration (Pressure Stabilization)

## Create `npt.mdp`

```bash
cat << EOF > npt.mdp
title                   = NPT equilibration
integrator              = md
nsteps                  = 50000
dt                      = 0.002

continuation            = yes
constraint_algorithm    = lincs
constraints             = h-bonds

cutoff-scheme           = Verlet
coulombtype             = PME
rcoulomb                = 1.0
rvdw                    = 1.0

tcoupl                  = V-rescale
tc-grps                 = RNA Water_and_ions
tau_t                   = 0.1 0.1
ref_t                   = 300 300

pcoupl                  = Parrinello-Rahman
pcoupltype              = isotropic
tau_p                   = 2.0
ref_p                   = 1.0
compressibility         = 4.5e-5

pbc                     = xyz
EOF
```

## Run NPT

```bash
gmx grompp -f npt.mdp -c nvt.gro -p topol.top -o npt.tpr -maxwarn 1
gmx mdrun -deffnm npt
```

✔ System is now thermodynamically equilibrated.

---
# 🔴 STEP 5 — Production MD (Scientific Simulation)

⚠️ **IMPORTANT NOTE**

The simulation time is controlled by the parameter:

```
nsteps
```

The total simulation time is calculated as:

```
Total time (ns) = (nsteps × dt) / 1000
```

In this workflow:

```
dt = 0.002 ps  (2 fs)
```

So:

```
1 ns = 500,000 steps
```

---

## 🧮 How to Set Simulation Length

| Desired Time | nsteps Required |
|-------------|-----------------|
| 15 ns       | 7,500,000       |
| 100 ns      | 50,000,000      |
| 250 ns      | 125,000,000     |
| 500 ns      | 250,000,000     |

---

## 📌 Example: 15 ns Simulation (Default)

```bash
cat << EOF > md.mdp
integrator      = md
dt              = 0.002
nsteps          = 7500000

nstxout-compressed = 1000
nstenergy          = 1000
nstlog             = 1000

continuation    = yes
constraints     = h-bonds
constraint_algorithm = lincs

cutoff-scheme   = Verlet
coulombtype     = PME
rcoulomb        = 1.0
rvdw            = 1.0

tcoupl          = V-rescale
tc-grps         = RNA Water_and_ions
tau_t           = 0.1 0.1
ref_t           = 300 300

pcoupl              = Parrinello-Rahman
pcoupltype          = isotropic
tau_p               = 2.0
ref_p               = 1.0
compressibility     = 4.5e-5

pbc             = xyz
EOF
```

---

## 🔬 If You Want 250 ns

Simply change:

```
nsteps = 125000000
```

---

## 🔬 If You Want 500 ns

Change to:

```
nsteps = 250000000
```

---

## 🚀 Run Production

```bash
gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr -maxwarn 1
gmx mdrun -deffnm md
```

---

## 📖 Scientific Recommendation

- 15 ns → short stability check  
- 100 ns → moderate conformational sampling  
- 250 ns → robust structural stability analysis  
- 500 ns → publication-level conformational study  

Longer simulations provide better conformational sampling for circRNA flexibility and intramolecular hydrogen bond stabilization.

---

# 📊 TRAJECTORY ANALYSIS

Before analysis, remove periodic boundary artifacts:

```bash
gmx trjconv -s md.tpr -f md.xtc -o md_noPBC.xtc -pbc mol -center
```

---

# 🔹 RMSD

```bash
gmx rms -s md.tpr -f md_noPBC.xtc -o rmsd.xvg
```

Select RNA group twice.

View:

```bash
xmgrace rmsd.xvg
```

Interpretation:
- Plateau → stable structure  
- Continuous drift → conformational change  

---

# 🔹 RMSF (Per Residue Fluctuation)

```bash
gmx rmsf -s md.tpr -f md_noPBC.xtc -o rmsf.xvg -res
```

View:

```bash
xmgrace rmsf.xvg
```

Interpretation:
- High RMSF at ends → flexible regions  
- Internal peaks → dynamic functional sites  

---

# 🔹 Radius of Gyration (Rg)

```bash
gmx gyrate -s md.tpr -f md.xtc -o gyrate.xvg
```

View:

```bash
xmgrace gyrate.xvg
```

Interpretation:
- Stable Rg → compact structure  
- Increasing Rg → expansion  

---

# 🔹 Hydrogen Bonds

```bash
gmx hbond -s md.tpr -f md.xtc -num hbnum.xvg
```

Select RNA for donor and acceptor.

View:

```bash
xmgrace hbnum.xvg
```

Optional occupancy map:

```bash
gmx hbond -s md.tpr -f md.xtc -hbm hbond_map.xpm
```

Interpretation:
- Stable H-bonds → conserved structure  
- Decreasing H-bonds → structural destabilization  

---

# 🔹 Contacts Analysis

```bash
gmx mindist -s md.tpr -f md.xtc -od mindist.xvg -on contacts.xvg
```

Select RNA twice.

View:

```bash
xmgrace contacts.xvg
```

Interpretation:
- More contacts → compact state  
- Fewer contacts → structural expansion  

---

# 📌 Final Outputs

- md.xtc  
- rmsd.xvg  
- rmsf.xvg  
- gyrate.xvg  
- hbnum.xvg  
- contacts.xvg  

---
# 🎥 Visualization of MD Trajectory in VMD

The molecular dynamics trajectory can be visualized as an animation (molecular movie) using VMD.

---

## 🧬 Software Required

VMD (Visual Molecular Dynamics)

Download:
https://www.ks.uiuc.edu/Research/vmd/

---

## 📂 Load the Structure and Trajectory

### Step 1 — Open VMD

Launch VMD from terminal:

```bash
vmd
```

---

### Step 2 — Load the Structure File

In VMD:

File → New Molecule  

Click **Browse** and load:

```
md.gro
```

or alternatively:

```
md.tpr
```

Click **Load**

---

### Step 3 — Load the Trajectory

With the molecule selected:

File → Load Data Into Molecule  

Browse and select:

```
md.xtc
```

Click **Load**

---

## ▶️ Play the Molecular Animation

Use the animation controls in the VMD main window:

- ▶ Play
- ⏸ Pause
- Adjust speed with the slider

This will display the full MD trajectory as a molecular movie.

---

## 🎨 Recommended Visualization Settings

Go to:

Graphics → Representations

Suggested settings:

- Drawing Method: NewCartoon
- Coloring Method: Name or ResName
- Material: Opaque

For better RNA visualization:

- Coloring Method → ResType
- Show hydrogen bonds (optional via HBonds plugin)

---

## 🎬 Export the Animation (Optional)

To create a movie file:

Extensions → Visualization → Movie Maker  

Select format (e.g., MPEG-4)  
Choose trajectory range  
Click **Make Movie**

---

## 🔬 Scientific Interpretation

Visualization allows:

- Structural stability assessment  
- Conformational transitions  
- Loop flexibility observation  
- Compaction/expansion detection  
- Hydrogen bond rearrangements  

For circRNA systems, animation helps identify:

- Backbone flexibility  
- Intramolecular contact persistence  
- Structural breathing motions 

# 👨‍🔬 Author

Luis Ernesto Castañeda Mota  
Department of Biochemistry  
Cinvestav-IPN  

---

# 📚 How to Cite This Work

If you use this workflow, please cite:

Castañeda-Mota, L.E.  
*circRNA Molecular Dynamics Simulation using GROMACS.*  
GitHub Repository (2026).

---

# 📜 License

MIT License
