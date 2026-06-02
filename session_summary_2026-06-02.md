# Session Summary — 2026-06-02

## 1. ENG-S8 (Subtilisin 16B) Trajectory Analysis

- Raw trajectories downloaded from MN5 (`~/glutenase_ff14SB`, 44 GB) to local machine
- Water/ions stripped using `createNonSolventTrajectories.py` (md_enzyme_analysis), variant `non_solvent`, Ca²⁺ retained
- Output: 15 stripped DCDs (5 × 100 ns chunks × 3 replicas) + `minimized_non_solvent.pdb`

**ENG-S8 active site:** Asp20 – His52 – Ser209 | P1 = Gln268 | Ca273, Ca274

---

## 2. Corrected Glutenase MD Analysis (All Systems)

All three S8 systems (WT-S8, MUT-S8, ENG-S8) re-analysed using `mfloor_analysis.py` with **stripped PDB as topology** — same approach as June 1. ENG-S8 added to the existing WT-S8/MUT-S8 analysis.

### Results (all replicas combined)

#### Catalytic triad distances
| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Asp–His mean (Å) | 4.98 ± 0.49 | 5.03 ± 0.71 | 5.32 ± 0.71 |
| Asp–His < 3.5 Å | 0.0% | 0.0% | 0.0% |
| His–Ser mean (Å) | 5.10 ± 0.98 | 5.75 ± 0.90 | 5.73 ± 1.29 |
| His–Ser < 3.5 Å | **8.8%** | 0.6% | 3.3% |

#### Tetrahedral attack geometry
| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Attack distance mean (Å) | 3.67 ± 0.59 | **3.29 ± 0.38** | 5.06 ± 0.76 |
| BD angle mean (°) | 59.6 ± 25.0 | **79.3 ± 11.2** | 46.7 ± 23.4 |
| Productive frames | 8.9% | **17.8%** | 0.2% |
| Best replica | rep1: 14.5% | rep3: **42.1%** | rep1: 0.6% |

#### Ca²⁺ coordination
| Ion | WT-S8 | MUT-S8 | ENG-S8 |
|-----|-------|--------|--------|
| Ca360 / Ca273 | 6.8 ± 0.4 | 6.7 ± 0.5 | 3.0 ± 0.1 |
| Ca361 / Ca274 | 7.6 ± 0.5 | 7.4 ± 0.5 | 6.3 ± 0.5 |
| Ca362 | 3.2 ± 1.4 | 3.0 ± 1.1 | — |

---

## 3. His Protonation Analysis

- All systems: **HID** (HD1 on ND1, NE2 free), pH 7.0
- Confirmed: **ND1 faces Asp, NE2 faces Ser** (canonical serine protease orientation, verified by VMD)
- propKa (protein-only, pH 7.0): Asp pKa 4.53/4.84/8.03 for ENG/WT/MUT; His pKa ~5.5 (unreliable for buried active site)
- **MUT-S8 Asp42 pKa 8.03** — strongly elevated, mutations increase burial; warrants re-examination

---

## 4. AF3 Jobs Submitted (New Simulations)

6 AF3 predictions submitted to MN5 (`acc_bscls`, array of 6):

| Job | Enzyme | Peptide | Ca²⁺ |
|-----|--------|---------|------|
| ENG-S8_LPYPQPQLP_ca | Subtilisin 16B (263 aa) | LPYPQPQLP | 2 |
| ENG-S8_QLPYPQPQL_ca | Subtilisin 16B (263 aa) | QLPYPQPQL | 2 |
| WT-S8_LPYPQPQLP_ca | Bga1903 (351 aa) | LPYPQPQLP | 3 |
| WT-S8_QLPYPQPQL_ca | Bga1903 (351 aa) | QLPYPQPQL | 3 |
| MUT-S8_LPYPQPQLP_ca | Bga1903 E219Q/S226L (351 aa) | LPYPQPQLP | 3 |
| MUT-S8_QLPYPQPQL_ca | Bga1903 E219Q/S226L (351 aa) | QLPYPQPQL | 3 |

Jobs folder: `marija_calculations/af3_eng_wt_mut_ca/`
SLURM script: `marija_calculations/slurm_array_af3_eng_wt_mut_ca.sh`

---

## 5. MD Setup Templates Prepared

Templates ready in `marija_calculations/md_new/` for new MD simulations (pH 8.5, ff14SB/TIP3P, 310 K, 3 × 500 ns — matching mfloor setup):
- `templates/tleap_template.in`
- `templates/submit_md_template.sh`
- `prepare_md_system.py` — auto-generates per-system files from AF3 best model PDB

---

## 6. Files

| File | Location |
|------|----------|
| Stripped ENG-S8 DCDs | `~/glutenase_ff14SB/ENG-S8/replica_0X/production_*_non_solvent.dcd` |
| Analysis results | `~/marija_calculations/mfloor_analysis_results/*.xlsx` |
| Plots | `~/marija_calculations/mfloor_analysis_results/plots/` |
| Report (md + docx) | `~/marija_calculations/mfloor_md_results_report.md/.docx` |
| AF3 jobs | `~/marija_calculations/af3_eng_wt_mut_ca/` |
| Sequences FASTA | `~/marija_calculations/eng_wt_mut_sequences.fasta` |
| MD templates | `~/marija_calculations/md_new/` |

---

## 7. Pending

- [ ] AF3 jobs running on MN5 — expected ~1.5 h once started
- [ ] Download AF3 results, select best model per system
- [ ] Run MD system preparation (propKa pH 8.5 → tleap → MN5 submission)
- [ ] Wait for subtilisin 16B MD job on MN5 (submitted 2026-06-01, ~5 days)
- [ ] MUT-S8 re-simulation with protonated Asp42 (pKa 8.03)
