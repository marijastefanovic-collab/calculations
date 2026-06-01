# MD Simulation Setup Report
## Subtilisin E + LPYPQPQLP Peptide + Ca²⁺

**Date:** 2026-06-01  
**Prepared by:** Marija Stefanovic  
**Cluster:** MareNostrum 5 (BSC), account `bsc72`

---

## 1. System Description

### Enzyme
- **Protein:** Mature subtilisin E (*Bacillus subtilis*)
- **Sequence source:** User-provided mature sequence (263 residues), starting at APALHSQ (signal peptide and propeptide removed)
- **Catalytic triad:** Asp20 – His52 – Ser209 (1-based biological numbering)
- **Note:** Full precursor sequence (382 aa) was tested with AlphaFold3 but produced poor predictions due to disordered signal peptide/propeptide; mature sequence was used for all final calculations

### Substrate
- **Peptide:** LPYPQPQLP (9 residues, chain B in AF3 model)
- **Predicted cleavage site:** Q5↓P6 (P1 = Gln5, based on AF3 distance analysis)
- **Basis:** OG(Ser209) – carbonyl C distance and ipTM scores across 25 AF3 models

### Cofactor
- **Ion:** Ca²⁺ (1 ion, CCD code CA)

### Starting Structure
- **Source:** AlphaFold3 prediction (seed-4_sample-0, best model)
- **Selection criteria:** Highest ranking score (0.75), ipTM = 0.69, OG–carbonyl C distance = 3.71 Å, P1 = Q5
- **Output format:** mmCIF → converted to PDB using gemmi

---

## 2. System Preparation

### 2.1 Protonation States (propKa 3.5.1, pH 8.5)

| Residue | pKa | State at pH 8.5 | AMBER name |
|---------|-----|-----------------|------------|
| Asp20 (catalytic) | 3.99 | Deprotonated | ASP |
| His52 (catalytic) | 7.62 | Neutral (pKa < 8.5) | HIE |
| His5 | 6.05 | Neutral | HIE |
| His27 | 6.18 | Neutral | HIE |
| His55 | 4.16 | Neutral | HIE |
| His214 | 4.65 | Neutral | HIE |
| His226 | 6.49 | Neutral | HIE |
| All Asp/Glu | < 6.0 | Deprotonated | ASP / GLU |

All histidine residues were set to **HIE** (neutral, protonated on Nε2) at pH 8.5. The catalytic His52 has pKa 7.62, placing it below pH 8.5 and therefore neutral at simulation pH.

### 2.2 tleap System Setup (AmberTools)

```
source leaprc.protein.ff14SB
source leaprc.water.tip3p

mol = loadpdb mature_subtilsin_ready.pdb
solvatebox mol TIP3PBOX 12.0 iso
addIons2 mol Na+ 0
addIons2 mol Cl- 0
saveamberparm mol mature_subtilsin.prmtop mature_subtilsin.inpcrd
```

| Parameter | Value |
|-----------|-------|
| Protein force field | ff14SB |
| Water model | TIP3P |
| Box type | Cubic |
| Box buffer | 12.0 Å |
| Neutralization | Na⁺/Cl⁻ to neutralize only |
| Water molecules | ~11,725 |
| Total atoms | ~38,000 |
| Box density (initial) | 0.842 g/cc |

**Files generated:**
- `mature_subtilsin.prmtop` — AMBER topology
- `mature_subtilsin.inpcrd` — coordinates

---

## 3. MD Simulation Protocol

### 3.1 Software

| Component | Version/Details |
|-----------|----------------|
| MD engine | OpenMM |
| Script | `openmm_simulation.py` (Martin Floor, BSC) |
| Platform | CUDA (GPU) |
| Precision | Mixed |
| Cluster | MareNostrum 5, `acc_bscls` partition |

### 3.2 Integrator

| Parameter | Value |
|-----------|-------|
| Integrator | Langevin Middle |
| Time step | 2 fs (0.002 ps) |
| Temperature | 310 K |
| Collision rate (γ) | 1 ps⁻¹ |
| Constraints | H-bonds (HBonds) |
| Rigid water | Yes |
| Non-bonded method | PME |
| Non-bonded cutoff | 1.0 nm (10 Å) |
| Ewald error tolerance | 1×10⁻⁴ |
| Barostat (NPT) | Monte Carlo, 1 bar |
| CMMotion remover | Every 100 steps |

### 3.3 Equilibration

| Stage | Duration | Details |
|-------|----------|---------|
| Energy minimization | Until convergence | All heavy atoms restrained (100 → 5 kcal/mol/Å²) |
| NVT heating | 0.1 ns (50,000 steps) | 5 K → 310 K, 50 temperature scaling steps |
| NPT equilibration | 0.5 ns (250,000 steps) | 310 K, 1 bar; restraint constant scaled from 5 → 0 kcal/mol/Å² over 50 steps |

**Positional restraints:** Applied to all heavy atoms (protein + peptide) during equilibration. Restraint constant: 5.0 kcal/mol/Å². Gradually released during NPT using a (1−t)⁴ scaling function.

### 3.4 Production

| Parameter | Value |
|-----------|-------|
| Total simulation time | 500 ns per replica |
| Chunk size | 100 ns (50,000,000 steps) |
| Number of chunks | 5 |
| DCD output interval | 100 ps (50,000 steps) |
| Data report interval | 100 ps |
| Frames per replica | 5,000 |
| Number of replicas | 3 |
| Total simulation time (all replicas) | 1,500 ns |
| Random seeds | 112358132, 471319856, 839215768 |

### 3.5 SLURM Job Parameters

```bash
#SBATCH --qos=acc_bscls
#SBATCH --time=48:00:00
#SBATCH --ntasks=1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=20
#SBATCH --account=bsc72
#SBATCH --array=1-3
```

---

## 4. Analysis Protocol

Analysis was performed using `md_enzyme_analysis` (Martin Floor, BSC) with a custom general-purpose script (`md_catalytic_analysis.py`).

### 4.1 Catalytic Geometry

| Metric | Description | Residues |
|--------|-------------|---------|
| Asp–His distance | Min distance Asp(OD1/OD2) → His(ND1) | Asp20, His52 |
| His–Ser distance | His(NE2) → Ser(OG) | His52, Ser209 |
| Attack distance | Ser(OG) → P1 carbonyl C | Ser209, Gln355 |
| Bürgi–Dunitz angle | Nu–C–O angle (productive: 90°–120°) | Ser209, Gln355 |
| Ca²⁺ coordination | First-shell coordination count, cutoff 2.9 Å | Ca²⁺ ions |

**Productive frame criterion:** attack distance < 4.0 Å AND 90° ≤ BD angle ≤ 120°

### 4.2 Config File (config_subtilsin.json)

```json
{
    "simulation_folder": "/home/ordi/marija_calculations/subtilsin_md",
    "output_dir": "/home/ordi/marija_calculations/subtilsin_analysis_results",
    "timestep_ps": 2.0,
    "traj_pattern": "production_*_non_water.dcd",
    "topology_pattern": "minimized_non_water.pdb",
    "triad": {"asp": 19, "his": 51, "ser": 208},
    "p1_resseq": 272,
    "ca_resseqs": [273]
}
```
*(resSeq values are 0-based as in the PDB file, i.e. 1 less than biological numbering)*

---

## 5. File Locations

| File | Location |
|------|---------|
| Mature subtilisin PDB (H-added) | `marija_calculations/mature_subtilsin_ready.pdb` |
| AMBER topology | `marija_calculations/mature_subtilsin.prmtop` |
| AMBER coordinates | `marija_calculations/mature_subtilsin.inpcrd` |
| tleap script | `marija_calculations/tleap_mature_subtilsin.in` |
| SLURM submission script | `marija_calculations/md/submit_subtilsin_md.sh` |
| OpenMM simulation script | `marija_calculations/mfloor_results/scripts/openmm_simulation.py` |
| Analysis script | `marija_calculations/md_catalytic_analysis.py` |
| Analysis config | `marija_calculations/config_subtilsin.json` |
| MN5 working directory | `/gpfs/projects/bsc72/marija_stefanovic/subtilsin_md/` |

---

## 6. Software References

- **OpenMM:** Eastman et al., *PLOS Comput. Biol.* 2017
- **AmberTools / ff14SB:** Maier et al., *J. Chem. Theory Comput.* 2015
- **TIP3P water:** Jorgensen et al., *J. Chem. Phys.* 1983
- **propKa:** Olsson et al., *J. Chem. Theory Comput.* 2011
- **MDTraj:** McGibbon et al., *Biophys. J.* 2015
- **AlphaFold3:** Abramson et al., *Nature* 2024
