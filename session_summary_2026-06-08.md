# Session Summary — 2026-06-08

## Work completed this session

---

### 1. 1 ns test replica analysis — all six systems

Full analysis of the existing 1 ns replicas (10 frames each) across all 6 systems.
Script: `/home/mstefano/projects/marija_calculations/plot_1ns_results.py`  
Report: `/home/mstefano/projects/reports/report_1ns_test_analysis.md`  
Figures: `/home/mstefano/projects/results/figures_1ns/`  
Data: `1ns_test_results/1ns_test_summary.xlsx`, `1ns_test_results/1ns_test_details.xlsx`

**Key results:**

| System | Attack d (Å) | BD angle (°) | Productive % | Oxy <3.5 Å % | His–Ser <3.5 Å % |
|--------|-------------|--------------|-------------|-------------|-----------------|
| **MUT-S8_QLPYPQPQL** | **3.17 ± 0.30** | **85.7 ± 7.0** | **30%** | **100%** | 0% |
| **WT-S8_LPYPQPQLP** | **3.32 ± 0.40** | **86.5 ± 5.8** | **20%** | **60%** | 30% |
| WT-S8_QLPYPQPQL | 5.13 ± 0.50 | 69.2 ± 11.2 | 0% | 0% | 0% |
| MUT-S8_LPYPQPQLP | 4.26 ± 0.46 | 46.7 ± 13.1 | 0% | 0% | 50% |
| ENG-S8_QLPYPQPQL | 4.22 ± 0.37 | 34.4 ± 6.8 | 0% | 0% | 40% |
| ENG-S8_LPYPQPQLP | 3.80 ± 0.28 | 27.9 ± 6.1 | 0% | 0% | 0% |

**Conclusions:**
- **MUT-S8_QLPYPQPQL** is the top performer: 3 productive frames, mean attack_d 3.17 Å, 100% oxyanion engagement. Only system with consistent near-attack geometry.
- **WT-S8_LPYPQPQLP**: 2 productive frames, 60% oxyanion engagement, tight Ca362 coordination (2.8 Å). Sampling catalytically relevant space but less consistently.
- **ENG-S8 (both peptides)**: confirmed non-productive. BD < 35° — Ser209-OG approaching carbonyl from the wrong face, forming H-bond with C=O oxygen instead of attacking C. Consistent with 100 ns ENG-S8_LPYPQPQLP findings.
- **MUT-S8_LPYPQPQLP**: good His–Ser contact (50% frames) but attack vector misaligned, BD 27–67°.
- **WT-S8_QLPYPQPQL**: peptide too far from Ser209, mean attack_d 5.13 Å.

---

### 2. Restrained simulations — attack distance (ATT)

**Purpose:** Apply an upper-wall harmonic restraint on the Ser-OG → P1-Gln-C distance to keep the substrate within attack range and test whether the remaining geometry (BD angle, oxyanion hole) spontaneously becomes productive.

**Restraint:** upper wall at 3.5 Å, k = 10 kcal/mol/Å², applied throughout production (not removed after equilibration).  
**Duration:** 1 ns, replica_01 only.  
**Script:** `/home/mstefano/projects/md 1 ns replicas ATT/scripts/openmm_simulation_att_restrained.py`  
**Upload script:** `/home/mstefano/projects/upload_md_1ns_att.sh` → MN5 `md_1ns_att/`

Systems prepared (6 total):

| System | Ser resid | P1-Gln resid | MN5 dir |
|--------|-----------|-------------|---------|
| ENG-S8_LPYPQPQLP | 208 | 267 | md_1ns_att/ENG-S8_LPYPQPQLP |
| ENG-S8_QLPYPQPQL | 208 | 268 | md_1ns_att/ENG-S8_QLPYPQPQL |
| MUT-S8_LPYPQPQLP | 287 | 355 | md_1ns_att/MUT-S8_LPYPQPQLP |
| MUT-S8_QLPYPQPQL | 287 | 356 | md_1ns_att/MUT-S8_QLPYPQPQL |
| WT-S8_LPYPQPQLP | 287 | 355 | md_1ns_att/WT-S8_LPYPQPQLP |
| WT-S8_QLPYPQPQL | 287 | 356 | md_1ns_att/WT-S8_QLPYPQPQL |

prmtop/inpcrd reused from existing `md_eng_wt_mut/` systems (already in `replica_01/input_files/`).

---

### 3. Restrained simulations — Bürgi–Dunitz angle (BD)

**Purpose:** Apply a lower-wall harmonic restraint on the Ser-OG···C=O Bürgi–Dunitz angle to prevent the non-productive collapse (BD < 70°) and test whether proper attack geometry is accessible.

**Restraint:** lower wall at 70°, k = 50 kcal/mol/rad², applied throughout production.  
**Duration:** 1 ns, replica_01 only.  
**Script:** `/home/mstefano/projects/md 1 ns replicas BD/scripts/openmm_simulation_bd_restrained.py`  
**Upload script:** `/home/mstefano/projects/upload_md_1ns_bd.sh` → MN5 `md_1ns_bd/`

Systems prepared (5 total — MUT-S8_LPYPQPQLP excluded as it showed no productive frames even with good His–Ser contact, deprioritised):

| System | Ser resid | P1-Gln resid | MN5 dir |
|--------|-----------|-------------|---------|
| ENG-S8_LPYPQPQLP | 208 | 267 | md_1ns_bd/ENG-S8_LPYPQPQLP |
| ENG-S8_QLPYPQPQL | 208 | 268 | md_1ns_bd/ENG-S8_QLPYPQPQL |
| MUT-S8_QLPYPQPQL | 287 | 356 | md_1ns_bd/MUT-S8_QLPYPQPQL |
| WT-S8_LPYPQPQLP | 287 | 355 | md_1ns_bd/WT-S8_LPYPQPQLP |
| WT-S8_QLPYPQPQL | 287 | 356 | md_1ns_bd/WT-S8_QLPYPQPQL |

prmtop/inpcrd reused from existing `md_eng_wt_mut/` systems.

---

### 4. AF3 complex_predictions_2.6 — new 1 ns test systems

Four models selected from the AF3 2.6 batch for unrestrained 1 ns test runs:

| System | AF3 source | Peptide | NATOM | Ca²⁺ (solvated) |
|--------|-----------|---------|-------|----------------|
| ENG-S8_LPYPQPQLP_s4p2 | seed-4, sample-2 | LPYPQPQLP | 39,913 | Ca273, Ca274 |
| ENG-S8_LPYPQPQLP_s5p4 | seed-5, sample-4 | LPYPQPQLP | 40,366 | Ca273, Ca274 |
| ENG-S8_QLPYPQPQL_s5p1 | seed-5, sample-1 | QLPYPQPQL | 39,625 | Ca273, Ca274 |
| MUT-S8_LPYPQPQLP_s1p1 | seed-1, sample-1 | LPYPQPQLP | 73,918 | Ca361, Ca362, Ca363 |

**Preparation pipeline** (`/home/mstefano/projects/md 1 ns replicas/prepare_new_2p6.py`):
- CIF → PDB via gemmi (Ca chains CAA/CAB/CAC renamed to C/D/E)
- propKa 3.5.1 at pH 8.5; catalytic His → HID (forced); others → HIE (all pKa < 8.5)
- All Ca²⁺ consolidated to chain C, unique sequential resids (prevents tleap "new molecule" drop)
- tleap: ff14SB + TIP3P + `frcmod.ions1lm_1264_tip3p` (12-6-4 Ca²⁺), 12 Å box

**tleap:** run locally with `/home/mstefano/micromamba/envs/tools/bin/tleap`; Errors = 0 for all 4; all Ca²⁺ ions confirmed present in solvated PDB.

**Upload:** rsync'd to MN5 `/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut/<system>/`.  
Local dirs: `/home/mstefano/projects/md 1 ns replicas/<system>/`

---

## Next steps

**Submit ATT and BD restrained runs** (upload first, then sbatch):
```bash
bash /home/mstefano/projects/upload_md_1ns_att.sh
bash /home/mstefano/projects/upload_md_1ns_bd.sh
# then on MN5:
for sys in ENG-S8_LPYPQPQLP ENG-S8_QLPYPQPQL MUT-S8_LPYPQPQLP MUT-S8_QLPYPQPQL WT-S8_LPYPQPQLP WT-S8_QLPYPQPQL; do
    cd /gpfs/projects/bsc72/marija_stefanovic/md_1ns_att/$sys && sbatch submit_${sys}_att_test.sh
done
for sys in ENG-S8_LPYPQPQLP ENG-S8_QLPYPQPQL MUT-S8_QLPYPQPQL WT-S8_LPYPQPQLP WT-S8_QLPYPQPQL; do
    cd /gpfs/projects/bsc72/marija_stefanovic/md_1ns_bd/$sys && sbatch submit_${sys}_bd_test.sh
done
```

**Submit new AF3 2.6 unrestrained runs** (already on MN5, pending queue space):
```bash
MN5_BASE=/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut
cd $MN5_BASE/ENG-S8_LPYPQPQLP_s4p2 && sbatch submit_ENG-S8_LPYPQPQLP_s4p2_test.sh
cd $MN5_BASE/ENG-S8_LPYPQPQLP_s5p4 && sbatch submit_ENG-S8_LPYPQPQLP_s5p4_test.sh
cd $MN5_BASE/ENG-S8_QLPYPQPQL_s5p1  && sbatch submit_ENG-S8_QLPYPQPQL_s5p1_test.sh
cd $MN5_BASE/MUT-S8_LPYPQPQLP_s1p1  && sbatch submit_MUT-S8_LPYPQPQLP_s1p1_test.sh
```

**After all 1 ns runs complete:**
- Verify NATOM in SLURM logs (OpenMM 12-6-4 Ca²⁺ check)
- Inspect in VMD; compare restrained vs. unrestrained geometry
- Monitor 100 ns production runs (job IDs 41509468–41509472) on MN5
