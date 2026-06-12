# Session Summary 2026-06-12

## Overview

Continued from previous session. Two main tasks completed:
1. Extended geometry analysis scripts finalised and run across all simulation types.
2. New 100 ns unrestrained submit scripts written for all 6 systems with corrected Ca²⁺ positions.

---

## 1. Extended Geometry Analysis (completed)

### Script: `marija_calculations/analyse_extended_geometry.py`

Computes 10 per-frame metrics for all simulation types (100 ns unrestrained, 1 ns unrestrained, 1 ns ATT, 1 ns BD, 1 ns ATT+BD):

| Column | Description |
|---|---|
| `ser_chi1_deg` | Catalytic Ser χ1 torsion (N–Cα–Cβ–OG) |
| `ser_dchi1_deg` | Frame-to-frame Δχ1, wrapped to [−180, 180] |
| `triad_rmsd_A` | RMSD of Asp/His/Ser heavy atoms vs frame 0 |
| `p1_n_gly_o_A` | P1 backbone N → S1-floor Gly backbone O distance |
| `p1_s1floor_min_A` | Min distance P1 heavy → all S1-floor heavy atoms |
| `s12_boundary_A` | Min distance S1/S2-boundary residue → peptide heavy atoms |
| `p1_oe1_asn_n_A` | P1 Gln OE1 → Asn backbone N (H-bond, S1 anchoring) |
| `p1_o_asn_nd2_A` | P1 backbone O → Asn ND2 (oxyanion hole pre-positioning) |
| `p1_ne2_gly_ca_A` | P1 Gln NE2 → Gly Cα |
| `p1_oe1_gly_ca_A` | P1 Gln OE1 → Gly Cα |

Key findings from 100 ns data:
- **MUT-S8_QLPYPQPQL**: oxyanion hole contact pre-formed in 99% of frames (`p1_o_asn_nd2` < 3.5 Å). Mechanistic explanation for highest productive frame count.
- **MUT-S8_LPYPQPQLP**: complementary anchoring contact (OE1→Asn-N) present in 89% of frames.
- **ENG-S8**: both anchoring contacts < 34% — poorly anchored P1 Gln relative to MUT/WT.
- **ENG-S8 Pro117 vs MUT/WT Gly191**: ENG-S8 S1/S2 boundary residue sits 0.5–1.7 Å closer to the peptide — explains non-productive Bürgi–Dunitz angles (Pro rigidity pushes peptide out of optimal trajectory).

### Extended geometry merged into filtered frames

Script `marija_calculations/filter_productive_frames.py` updated to merge `100ns_extended.xlsx` into `filtered_frames.xlsx`. Output now has 20 columns (original 10 core metrics + 10 extended geometry).

### Output files

| File | Description |
|---|---|
| `results/100ns_results/100ns_extended.xlsx` | 10,000 frames, 10 metrics |
| `results/1ns_unrestrained_results/1ns_unrest_extended.xlsx` | 1 ns unrestrained |
| `results/att_1ns_results/att_1ns_extended.xlsx` | 1 ns ATT |
| `results/bd_1ns_results/bd_1ns_extended.xlsx` | 1 ns BD |
| `results/att_bd_1ns_results/att_bd_1ns_extended.xlsx` | 1 ns ATT+BD |
| `results/100ns_results/filtered_frames.xlsx` | Regenerated with 20 columns |

---

## 2. 100 ns Unrestrained Resubmission (corrected Ca²⁺)

### Motivation

Previous 100 ns simulations (`md_eng_wt_mut`) used prmtop files with wrong Ca²⁺ positions. Corrected prmtop/inpcrd files exist in the ATT simulation directory on MN5 (`md_100ns_att`).

### New scripts: `md_100ns_unrest/`

Six SLURM array scripts created, one per system:

| System | Script |
|---|---|
| ENG-S8_LPYPQPQLP | `md_100ns_unrest/ENG-S8_LPYPQPQLP/submit_ENG-S8_LPYPQPQLP_100ns.sh` |
| ENG-S8_QLPYPQPQL | `md_100ns_unrest/ENG-S8_QLPYPQPQL/submit_ENG-S8_QLPYPQPQL_100ns.sh` |
| MUT-S8_LPYPQPQLP | `md_100ns_unrest/MUT-S8_LPYPQPQLP/submit_MUT-S8_LPYPQPQLP_100ns.sh` |
| MUT-S8_QLPYPQPQL | `md_100ns_unrest/MUT-S8_QLPYPQPQL/submit_MUT-S8_QLPYPQPQL_100ns.sh` |
| WT-S8_LPYPQPQLP  | `md_100ns_unrest/WT-S8_LPYPQPQLP/submit_WT-S8_LPYPQPQLP_100ns.sh`  |
| WT-S8_QLPYPQPQL  | `md_100ns_unrest/WT-S8_QLPYPQPQL/submit_WT-S8_QLPYPQPQL_100ns.sh`  |

### Parameters

- `--qos=acc_bscls`, `--time=24:00:00`, `--array=1-3`
- 100 ns production, single chunk, no restraints (`openmm_simulation.py`)
- prmtop/inpcrd: absolute paths to `/gpfs/projects/bsc72/marija_stefanovic/md_100ns_att/{system}/replica_01/input_files/`
- Script: absolute path to `/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut/{system}/scripts/openmm_simulation.py`
- Cluster output: `/gpfs/projects/bsc72/marija_stefanovic/md_100ns_unrest/{system}/replica_0N/`
- Seeds: rep1=112358132, rep2=471319856, rep3=839215768 (consistent with ATT runs)

### Cluster preparation

Working directories created on MN5. Upload script: `upload_md_100ns_unrest.sh`.

To submit after uploading:
```bash
bash /home/mstefano/projects/upload_md_100ns_unrest.sh
# then on MN5, cd into each system dir and: sbatch submit_{system}_100ns.sh
```

---

## Pending

- Upload scripts and submit jobs to MN5
- Wait for 100 ns unrestrained simulations to complete (~24 h per job)
- Wait for 100 ns ATT simulations (already submitted previously)
- Download and analyse new 100 ns trajectories when done
- Clustering of productive frames (planned)
