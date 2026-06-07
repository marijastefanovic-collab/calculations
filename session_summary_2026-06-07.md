# Session Summary — 2026-06-07

## Work completed this session

### 1. ENG-S8_LPYPQPQLP 100 ns analysis
Full analysis pipeline run on 3 replicas × 100 ns (1000 frames each):
- Catalytic triad geometry (Asp20–His52–Ser209)
- Tetrahedral attack geometry (Ser209-OG → Gln-C, Bürgi-Dunitz angle)
- Ca²⁺ coordination at 2.9 Å and 3.2 Å
- Oxyanion hole (Ser209-N → Gln-O)
- Peptide RMSD (heavy + backbone, vs. minimized reference)
- Protein backbone RMSD (whole protein + active site)

Results: `/home/mstefano/projects/marija_calculations/eng_s8_100ns_results/`

**Key finding:** ENG-S8_LPYPQPQLP is fully non-productive across all 3 replicas and 300 ns total.
Root cause traced to AF3 starting structure: BD angle 134.9° (OG approaching from wrong side of carbonyl).
During equilibration OG swings to form H-bond with carbonyl O instead of attacking C → BD collapses to ~39°.
Simulation quality is good (protein RMSD ~1 Å, active site RMSD <0.7 Å) — non-productive geometry is real.

### 2. BD angle added to AF3 results tables
Computed Bürgi-Dunitz angle for all AF3 best structures and added to:
- `AF3_analysis_all_models 2.06.xlsx` — all 150 models (25 per system)
- `AF3_best_GLN_P1 2.06.xlsx` — best model per complex
- `best_structure_selection_bd_corrected.xlsx` — final selection table

MUT-S8_QLPYPQPQL was missing from both files; all 25 models parsed from CIF and added.

**Best structures (final selection):**
| System | Ser–C dist | BD angle | Productive |
|--------|-----------|----------|------------|
| ENG-S8_QLPYPQPQL | 3.6 Å | 115.6° | ✓ |
| MUT-S8_LPYPQPQLP | 3.9 Å | 114.8° | dist >4 |
| ENG-S8_LPYPQPQLP | 4.1 Å | 134.9° | ✗ |
| WT-S8_QLPYPQPQL | 3.8 Å | 104.5° | dist >4 |
| WT-S8_LPYPQPQLP | 3.5 Å | 108.1° | ✓ |
| MUT-S8_QLPYPQPQL | 3.1 Å | 101.0° | ✓ |

### 3. 1 ns test runs — all 6 systems analysed
Downloaded and analysed 1 ns replicas for all 6 systems (10 frames each).
VMD visualization scripts written for all 6: `vmd_1ns_{system}.tcl`

**Standout: MUT-S8_QLPYPQPQL** — attack_d = 3.17 Å, BD = 85.7°, 30% productive frames, 100% oxyanion engagement.

### 4. 100 ns 3-replica jobs submitted for 5 systems
Systems: ENG-S8_QLPYPQPQL, WT-S8_LPYPQPQLP, WT-S8_QLPYPQPQL, MUT-S8_LPYPQPQLP, MUT-S8_QLPYPQPQL

Before submitting:
- Production time corrected: 500 ns → 100 ns in all submit scripts
- Ca²⁺ counts verified in prmtop (2 for ENG-S8, 3 for WT/MUT-S8)
- `input_files/` (prmtop + inpcrd) copied to replica_02 and replica_03 for 4 systems that were missing them

| System | Job ID |
|--------|--------|
| ENG-S8_QLPYPQPQL | 41509468 |
| WT-S8_LPYPQPQLP | 41509469 |
| WT-S8_QLPYPQPQL | 41509470 |
| MUT-S8_LPYPQPQLP | 41509471 |
| MUT-S8_QLPYPQPQL | 41509472 |

SLURM: acc_bscls queue, 48h, 1 GPU, array 1-3.

## Next steps
- Monitor jobs (check with `squeue -u bsc72` on MN5)
- When complete: download stripped trajectories and run analysis for all 5 systems
- Compare all 6 systems side by side
