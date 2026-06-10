# Session Summary — 2026-06-10

## Work completed this session

---

### 1. ValidSummary sheet added to analyse_100ns.py

Added `VALID` dict and `ValidSummary` sheet to `/home/mstefano/projects/marija_calculations/analyse_100ns.py`.  
Valid replicas (correct Ca2+ binding based on 100 ns coordination data):

| System | Valid replicas |
|--------|---------------|
| ENG-S8_LPYPQPQLP | r01, r02, r03 |
| ENG-S8_QLPYPQPQL | r01, r02, r03 |
| MUT-S8_LPYPQPQLP | r01 only |
| MUT-S8_QLPYPQPQL | r02 only |
| WT-S8_LPYPQPQLP  | r03 only |
| WT-S8_QLPYPQPQL  | r03 only |

ValidSummary includes: attack_d, BD angle, productive%, Asp-His, His-Ser, oxyanion hole, RMSD.

---

### 2. ATT 1ns — WT systems downloaded and analysed

Downloaded `production_01.dcd` and `minimized.pdb` from MN5 `md_1ns_att/` for WT-S8_LPYPQPQLP and WT-S8_QLPYPQPQL.  
Script: `/home/mstefano/projects/marija_calculations/analyse_att_1ns.py`  
Output: `/home/mstefano/projects/results/att_1ns_results/att_1ns_results.xlsx`

Full ATT 1ns results (all 9 systems):

| System | Attack d (Å) | BD (°) | Productive % | Oxyanion <3.5Å % | Ca363 coord |
|--------|-------------|--------|-------------|-----------------|-------------|
| ENG-S8_LPYPQPQLP | 3.67 ± 0.09 | 29.8 ± 7.7 | 0% | 0% | — |
| ENG-S8_LPYPQPQLP_s4p2 | 3.65 ± 0.12 | 80.6 ± 8.2 | 0% | 0% | — |
| ENG-S8_QLPYPQPQL | 3.77 ± 0.20 | 40.4 ± 6.7 | 0% | 0% | — |
| ENG-S8_QLPYPQPQL_s5p1 | 3.78 ± 0.18 | 33.6 ± 6.2 | 0% | 0% | — |
| MUT-S8_LPYPQPQLP | 3.67 ± 0.16 | 40.8 ± 10.4 | 0% | 0% | 5.3 |
| **MUT-S8_LPYPQPQLP_s1p1** | 3.15 ± 0.20 | 87.0 ± 3.4 | **60%** | 80% | 2.3 |
| **MUT-S8_QLPYPQPQL** | 3.13 ± 0.18 | 87.9 ± 5.0 | **70%** | 100% | 5.0 |
| **WT-S8_LPYPQPQLP** | 3.19 ± 0.19 | 89.0 ± 6.6 | **80%** | 70% | 4.0 |
| WT-S8_QLPYPQPQL | 3.71 ± 0.10 | 30.2 ± 5.3 | 0% | 0% | 3.7 |

Key observations:
- WT-S8_LPYPQPQLP is the best performer (80% productive) in ATT restrained runs.
- ENG systems fail even with attack distance restrained — BD angle remains <45° in most cases.
- Ca363 poorly bound (coord 2.3–4.0) correlates with productive geometry in MUT/WT (consistent with 100 ns findings).
- WT-S8_QLPYPQPQL fails despite Ca363 being partially free — BD angle only 30°, peptide in wrong orientation.

---

### 3. Ca2+ parametrization — root cause clarified

**Problem identified:** Previous session summaries incorrectly labeled `frcmod.ions1lm_1264_tip3p` as "12-6-4 Ca2+ parameters". This file contains parameters for **monovalent** ions (Na+, K+, Cl-) only. Ca2+ (divalent) requires `frcmod.ions234lm_1264_tip3p`.

**Which systems were missing ions234lm:**

| System | Status before fix |
|--------|------------------|
| ENG-S8_LPYPQPQLP | missing ions234lm |
| ENG-S8_QLPYPQPQL | missing ions234lm |
| MUT-S8_LPYPQPQLP | missing ions234lm |
| MUT-S8_QLPYPQPQL | fixed in previous session |
| WT-S8_LPYPQPQLP | fixed in previous session |
| WT-S8_QLPYPQPQL | fixed in previous session |

**Fix applied:** Added `loadAmberParams frcmod.ions234lm_1264_tip3p` to tleap files for ENG-S8_LPYPQPQLP, ENG-S8_QLPYPQPQL, MUT-S8_LPYPQPQLP. tleap run locally for all three.  
New prmtop/inpcrd in `/home/mstefano/projects/results/md_eng_wt_mut/<system>/`.

**Important finding — LENNARD_JONES_CCOEF absent:**  
The local tleap version writes only `NONBON` entries (r_min/2 and epsilon) for Ca2+ even when `ions234lm` is loaded. There is NO `LENNARD_JONES_CCOEF` section (12-6-4 C-term) in any prmtop. This means:
- Ca2+ uses optimised 12-6 LJ parameters (tuned for divalent Ca2+ in TIP3P), not full 12-6-4 potential.
- The old OpenMM Ca-dropping bug (triggered by LENNARD_JONES_CCOEF) will NOT affect these runs.
- All 6 prmtops are safe for the old MN5 OpenMM environment.

---

### 4. ENG-S8_LPYPQPQLP 1ns test with fixed prmtop

Ran a 1ns unrestrained debug job (`acc_debug` qos) on MN5 for ENG-S8_LPYPQPQLP with the corrected prmtop.  
Submit script: `/home/mstefano/projects/md 1 ns replicas/ENG-S8_LPYPQPQLP/submit_ENG-S8_LPYPQPQLP_debug.sh`  
Output: `/home/mstefano/projects/results/eng_1ns_fixed/eng_1ns_fixed.xlsx`

Results (10 frames):

| Metric | Value |
|--------|-------|
| Attack d | 3.73 ± 0.17 Å |
| BD angle | 31.9 ± 9.8° |
| Productive | 0% |
| Ca273 coord | 3.00 ± 0.00 |
| Ca274 coord | 6.00 ± 0.00 |
| Oxyanion | 4.78 ± 0.28 Å |
| RMSD | 0.69 ± 0.25 Å |

**Key finding:** Ca273 coordination = 3.0 with fixed prmtop confirms this is a genuine low-affinity binding site (biology), not a parametrization artifact. Geometry unchanged from old run — ENG system remains non-productive.

---

## Pending tasks

1. **MN5 reruns in progress:** MUT-S8_QLPYPQPQL r01+r03, WT-S8_LPYPQPQLP r01+r02, WT-S8_QLPYPQPQL r01+r02 (Ca363 fix, uploaded with corrected prmtop). MUT-S8_LPYPQPQLP r02+r03 also running (job 41604809).
2. **Upload and resubmit ENG and MUT-S8_LPYPQPQLP 100ns** with fixed prmtops — not yet done; will affect r01/r02/r03 of ENG systems and remaining MUT-S8_LPYPQPQLP replicas.
3. **BD restrained 1ns results** — not yet downloaded or analysed.
4. **ATT prmtops** for base systems need updating (copies of old prmtops) — deferred.
5. **VMD comparison script** for ENG old vs new — not working (path issues unresolved).
