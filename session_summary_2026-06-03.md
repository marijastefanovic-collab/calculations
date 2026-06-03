# Session Summary — 2026-06-03

## 1. Environment Setup (Local Machine)

Full computational environment configured on the BSC workstation:

- **Python venv** at `~/projects/.venv` (Python 3.13): prepare_proteins, bsc_calculations, md_enzyme_analysis, gemmi, propKa, biopython, mdtraj, pandas, numpy, scipy, matplotlib, seaborn
- **Micromamba `tools` env**: blast 2.17.0, mafft 7.526, cd-hit 4.8.1, mkdssp 4.6.1, entrez-direct 24.0, AmberTools 25, PyMOL 3.1.0
- **VMD 2.0.0** installed at `~/.local/bin/vmd`
- dssp symlink: `~/.local/bin/dssp → mkdssp`

---

## 2. MD Setup — ENG-S8, WT-S8, MUT-S8 (AF3 Best Models)

### 2.1 Model Preparation

Best AF3 models (selected with GLN at P1) converted from CIF to PDB using gemmi 0.7.5:

| System | Seed | Sample | ipTM | Ser–C (Å) | Scissile Bond |
|--------|------|--------|------|-----------|---------------|
| ENG-S8_LPYPQPQLP | 5 | 3 | 0.69 | 3.55 | GLN5-PRO6 |
| ENG-S8_QLPYPQPQL | 3 | 0 | 0.71 | 3.83 | GLN6-PRO7 |
| WT-S8_LPYPQPQLP  | 1 | 1 | 0.78 | 3.48 | TYR3-PRO4 |
| WT-S8_QLPYPQPQL  | 4 | 3 | 0.73 | 2.83 | GLN8-LEU9 |
| MUT-S8_LPYPQPQLP | 3 | 2 | 0.75 | 3.82 | TYR3-PRO4 |

MUT-S8_QLPYPQPQL excluded — AF3 job timed out (see Section 3).

### 2.2 propKa Analysis (pH 8.5)

propKa 3.5.1 run on all 5 systems. Full report: `propka_report_2026-06-03.md`.

| System | Catalytic Asp pKa | Catalytic His pKa | Assignment |
|--------|------------------|------------------|------------|
| ENG-S8_LPYPQPQLP | Asp20 = 4.02 | His52 = 7.63 | Asp deprotonated, His52 → HID |
| ENG-S8_QLPYPQPQL | Asp20 = 4.07 | His52 = 7.45 | Asp deprotonated, His52 → HID |
| MUT-S8_LPYPQPQLP | Asp42 = 4.37 | His116 = 7.46 | Asp deprotonated, His116 → HID |
| WT-S8_LPYPQPQLP  | Asp42 = 4.34 | His116 = 7.16 | Asp deprotonated, His116 → HID |
| WT-S8_QLPYPQPQL  | Asp42 = 4.33 | His116 = 7.54 | Asp deprotonated, His116 → HID |

**Histidine assignment rule:** Catalytic His (His52 ENG-S8 / His116 WT+MUT) → **HID** (ND1 protonated, faces Asp; NE2 free, faces Ser). All other His assigned from propKa: pKa < 8.5 → HIE, pKa > 8.5 → HIP. All non-catalytic His in all systems had pKa < 8.5 → HIE.

### 2.3 System Preparation (tleap — AmberTools 25, local)

tleap run locally for all 5 systems. Parameters matching mfloor reference pipeline:
- Force field: ff14SB + TIP3P
- Box: cubic, 12 Å buffer
- Neutralisation only (no extra salt)
- Result: **Errors = 0** for all systems (minor close-contact warnings from AF3 model, resolved by energy minimisation)

### 2.4 SLURM Submission Scripts

- Partition: `acc_bscls`, 48h wall time
- 3-replica array (seeds: 112358132, 471319856, 839215768)
- 500 ns production (5 × 100 ns chunks), 310 K, 2 fs timestep
- NVT 0.1 ns (5→310 K), NPT 0.5 ns (restraint 5 kcal/mol/Å² → 0)

All files uploaded to MN5: `/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut/`

**5 MD jobs submitted to MN5 (acc_bscls) — pending in queue as of end of session.**

---

## 3. MUT-S8_QLPYPQPQL AF3 Error — Root Cause & Fix

**Root cause:** TIME LIMIT. The job ran as task 6 in a 6-task SLURM array. By the time it reached MUT-S8_QLPYPQPQL, the MSA search for the 351aa sequence took 36 minutes (vs 8 min for ENG-S8's 263aa), and template deduplication was still running when the wall time was reached (~2h into that task).

```
slurmstepd: error: JOB 41365441 CANCELLED AT 2026-06-02T18:40:30 DUE TO TIME LIMIT
```

The input JSON was correct — identical structure to the working MUT-S8_LPYPQPQLP job.

**Fix:** Resubmitted as standalone task 6:
```bash
sbatch --array=6 slurm_array_af3_eng_wt_mut_ca.sh
```
Job submitted 2026-06-03, pending in queue.

---

## 4. Subtilisin E MD — Trajectory Stripping

Subtilisin E MD simulation (3 replicas × 500 ns) completed on MN5. Trajectories located at:
`/gpfs/projects/bsc72/marija_stefanovic/md/runs/subtilsin_md/`

Stripping script (`strip_subtilsin.py`) uploaded and submitted to MN5 (acc_bscls). Removes WAT, Na+, Cl−; retains protein, peptide, Ca²⁺ (Ca273, Ca274).

**Stripping job submitted — pending in queue as of end of session.**

Once complete: download stripped DCDs → run `subtilsin_analysis.py` locally.

---

## 5. Analysis Script — Subtilisin E

Written: `marija_calculations/subtilsin_analysis.py`

Implements the standard 4-metric pipeline using `md_enzyme_analysis`:
1. **Catalytic triad geometry** — Asp20(OD)↔His52(ND1) and His52(NE2)↔Ser209(OG), threshold < 3.5 Å, HID tautomer
2. **Tetrahedral attack geometry** — Ser209(OG)→Gln5(C) distance + Bürgi–Dunitz angle; productive: < 4.0 Å AND 90–120°
3. **Ca²⁺ coordination** — Ca273 and Ca274, first-shell cutoff 2.9 Å, protein O/N donors only
4. **Oxyanion formation** — Ser209(N) backbone NH → Gln5(O) carbonyl distance

Output: `subtilsin_analysis_results/` (triad_geometry.xlsx, tetrahedral_attack.xlsx, ca_coordination.xlsx, oxyanion.xlsx)

---

## 6. Pending Tasks

- [ ] MUT-S8_QLPYPQPQL AF3 job — wait for result, then set up MD
- [ ] Subtilisin E trajectory stripping — wait for MN5 job
- [ ] Download stripped subtilisin DCDs → run analysis
- [ ] Monitor 5 new MD jobs (ENG/WT/MUT-S8) — expected ~5 days
- [ ] After MD: run full 4-metric analysis for all systems
- [ ] Write subtilisin E MD analysis report
- [ ] Write ENG/WT/MUT-S8 MD analysis report (comparative)
