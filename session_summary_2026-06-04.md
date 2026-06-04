# Session Summary — 2026-06-04

## 1. MN5 Cleanup

- Deleted all `.err` and `.out` SLURM log files from AF3 jobs
- Deleted `af3/` and `af3_eng_wt_mut_ca/` AF3 result directories (all downloaded locally)
- Deleted `md_eng_wt_mut/` (29 GB) — cancelled jobs with Ca²⁺ setup error
- Deleted solvated DCDs from `md/runs/subtilsin_md/` — stripped trajectories already downloaded

---

## 2. Subtilisin 16B (ENG-S8) MD Analysis

- Stripped trajectories downloaded from MN5 (3 replicas × 5 chunks)
- Fixed paths in `subtilsin_analysis.py`: BASE → `md/`, DCD path without `stripped/` subdir, removed Ca274 (only 1 Ca in system)
- **Analysis run:** 15,000 frames (3 × 500 ns equivalent)

### Results

| Metric | ENG-S8 |
|--------|--------|
| Asp–His < 3.5 Å | 0.0% |
| His–Ser < 3.5 Å | 2.0% |
| Attack distance mean (Å) | 5.27 ± 0.73 |
| BD angle mean (°) | 63.7 ± 18.6 |
| Productive frames | 0.0% |
| Oxyanion < 3.5 Å | 0.0% |
| Ca272 coordination | 6.8 ± 0.4 (stable) |

- Generated 6 publication-ready plots → `subtilsin_analysis_results/plots/`
- Written report → `subtilsin_md_results_report.md` (pushed to GitHub)

---

## 3. Ca²⁺ Setup Bug Identified and Fixed

**Problem:** Both Ca²⁺ ions in ENG-S8_LPYPQPQLP had `resid 1` in the AF3 PDB (from chains C and D), causing tleap to merge them into one residue with duplicate atoms. Only 1 Ca²⁺ survived into the simulation. Same issue existed in all systems.

**Fix:**
- Added `loadAmberParams frcmod.ions1lm_1264_tip3p` to all tleap files (proper Ca²⁺ force field parameters for TIP3P)
- Modified `prepare_systems.py` → `prepare_pdb()` to renumber Ca²⁺ ions with unique sequential resSeq:
  - ENG-S8: Ca273, Ca274
  - WT-S8/MUT-S8: Ca360, Ca361, Ca362

---

## 4. MUT-S8_QLPYPQPQL System Added

- AF3 analysis: 25 models analysed, best model **seed-2_sample-1** (ranking 0.815, ipTM 0.76, P1=Q6, Ser–Q6 dist=3.06 Å, His-Ser=2.98 Å)
  - Scissile bond: Q6–P7
  - Results saved → `af3_MUT-S8_QLPYPQPQL_distances.xlsx`
- CIF → PDB conversion (gemmi, Ca chain renaming)
- propKa at pH 8.5 (chain A only, no ligand):
  - His116: pKa 8.19 → **HID (forced)**
  - His48: pKa 5.94 → HIE; His119: 3.56 → HIE; His293: 3.99 → HIE
  - Asp42: pKa 4.11 (normal, deprotonated at pH 8.5)
- propKa report → `/home/mstefano/projects/MUT-S8_QLPYPQPQL_propka_report.md`
- Added to `prepare_systems.py`, generated `_ready.pdb`, tleap input, SLURM scripts

---

## 5. MD Preparation — All 6 Systems

All systems regenerated with Ca²⁺ fix and correct HID:

| System | Cat. His | Ca²⁺ |
|--------|----------|------|
| ENG-S8_LPYPQPQLP | His52 HID | Ca273, Ca274 |
| ENG-S8_QLPYPQPQL | His52 HID | Ca273, Ca274 |
| MUT-S8_LPYPQPQLP | His116 HID | Ca360, Ca361, Ca362 |
| MUT-S8_QLPYPQPQL | His116 HID | Ca360, Ca361, Ca362 |
| WT-S8_LPYPQPQLP | His116 HID | Ca360, Ca361, Ca362 |
| WT-S8_QLPYPQPQL | His116 HID | Ca360, Ca361, Ca362 |

tleap run locally (`/home/mstefano/micromamba/envs/tools/bin/tleap`) — 0 errors for all systems.

---

## 6. Test Jobs Submitted (1 ns, replica 1)

Two test jobs submitted to MN5 (acc_bscls) to verify Ca²⁺ coordination and HID conformation:

| Job | SLURM ID | Status |
|-----|----------|--------|
| ENG-S8_LPYPQPQLP test | 41434642 | Pending |
| MUT-S8_LPYPQPQLP test | — | Pending |

Scripts: `submit_ENG-S8_LPYPQPQLP_test.sh`, `submit_MUT-S8_LPYPQPQLP_test.sh`

---

## 7. PyMOL Guidelines Written

- `pymol_publication_guidelines.md` — color palette, representations, ray tracing, export sizes
- `pymol_representations_guide.md` — pLDDT coloring, surface, B-factor, electrostatics, etc.
- `pymol_template_active_site.pml` — ready-to-use PyMOL script template

All saved to `/home/mstefano/projects/`

---

## 8. Pending

- [ ] ENG-S8_LPYPQPQLP and MUT-S8_LPYPQPQLP test jobs (1 ns) — verify Ca²⁺ coordination and HID in VMD
- [ ] If test OK: run tleap for remaining 4 systems and submit full 500 ns × 3 replicas for all 6
- [ ] MUT-S8_QLPYPQPQL: tleap not yet run (only MUT-S8_LPYPQPQLP and ENG-S8_LPYPQPQLP done today)
- [ ] Remaining 4 systems still need Ca²⁺-fixed tleap run
- [ ] MUT-S8 re-simulation with protonated Asp42 (from June 2 pending list — Asp42 pKa 8.03 in LPYPQPQLP model; pKa 4.11 in QLPYPQPQL model, likely not needed for that one)
