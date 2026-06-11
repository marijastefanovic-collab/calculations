# Session Summary — 2026-06-11

## Work completed this session

---

### 1. Ca2+ parameterization — definitive root cause identified

**Why `frcmod.ions234lm_1264_tip3p` was never loading:**

`leaprc.water.tip3p` loads `frcmod.ions234lm_126_tip3p` as its last line. When the tleap input subsequently calls `loadAmberParams frcmod.ions234lm_1264_tip3p`, tleap silently skips it because all divalent/higher ion atom types (Ca2+, Mg2+, Zn2+, etc.) are already defined. The "Loading parameters:" message never appears for the 1264 file in any leap log. This is confirmed by inspecting `leap_MUT-S8_QLPYPQPQL.log`: the log jumps directly from loading `frcmod.ions1lm_1264_tip3p` (monovalent) to loading the PDB, with no entry for `frcmod.ions234lm_1264_tip3p`.

**Additionally:** `frcmod.ions234lm_1264_tip3p` in the local tools conda env contains only `NONBON` entries (R_min/2, epsilon). It has no `LJEDIT` section with C4 coefficients. True 12-6-4 Ca2+ requires `LJEDIT` entries in the frcmod and a `LENNARD_JONES_CCOEF` section in the prmtop, which `saveamberparm` in this tleap version does not write. This means:

- The 12-6-4 Ca2+ model is not achievable via `loadAmberParams` with this tleap installation, regardless of file load order.
- All prmtops use 12-6 Li/Merz Ca2+ parameters (from `frcmod.ions234lm_126_tip3p`, Li et al. JCTC 2013) consistently across all 6 systems.
- Ca2+ R_min/2: 1.649 Å, eps: 0.10593 kcal/mol (126 set vs 1.642 / 0.10186 for 1264 — difference is small and within normal parameter uncertainty).
- Relative comparisons across systems remain valid.

**Consequence for the fix applied on Jun 10:** The re-generated prmtops for ENG-S8_LPYPQPQLP, ENG-S8_QLPYPQPQL, and MUT-S8_LPYPQPQLP are correct in all other respects (proper ion counts, correct solvation) and are consistent with all other systems. The Ca2+ parameters are the same as in the other three systems.

---

### 2. md_100ns_att prmtops verified

All 18 prmtops in `md_100ns_att/` (6 systems × 3 replicas) were verified by file size to match the fixed prmtops from `results/md_eng_wt_mut/` (generated Jun 10). No stale copies found.

---

## Pending tasks

1. **Upload and submit ENG + MUT-S8_LPYPQPQLP 100ns unrestrained** with fixed prmtops to MN5 — not yet done.
2. **Upload and submit md_100ns_att** (100ns ATT-restrained, 6 systems × 3 replicas) to MN5 — not yet done.
3. **Download and analyse BD restrained 1ns results** — DCDs not yet retrieved from MN5.
4. **Update ATT 1ns prmtops** for variant systems (ENG_s4p2, ENG_s5p1, MUT_s1p1) — still using pre-fix prmtops.
5. **MN5 reruns in progress** (from Jun 10): MUT-S8_QLPYPQPQL r01+r03, WT-S8_LPYPQPQLP r01+r02, WT-S8_QLPYPQPQL r01+r02, MUT-S8_LPYPQPQLP r02+r03 — await completion and re-analysis.
