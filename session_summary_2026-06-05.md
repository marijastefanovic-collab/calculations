# Session Summary — 2026-06-05

## 1. Verified 1 ns test replicas (ENG-S8_LPYPQPQLP and MUT-S8_LPYPQPQLP)

### MUT-S8_LPYPQPQLP — clean
- All 73,195 atoms present in simulation
- 3 Ca²⁺ ions (Ca361, Ca362, Ca363) all coordinated to protein
- Catalytic triad Asp42–His116–Ser288 geometry confirmed good in VMD
- No issues

### ENG-S8_LPYPQPQLP — problem found and fixed
**Problem:** First run (job 41427873) used an older OpenMM version on MN5 that silently dropped 18 atoms: Ca274 (one of two structural Ca²⁺), 2 Cl⁻ counterions, and 5 water molecules. System ran with 40,330 atoms instead of 40,348. Only Ca273 was present in the simulation.

**Root cause:** Older OpenMM could not handle the 12-6-4 LJ parameters (`frcmod.ions1lm_1264_tip3p`, `LENNARD_JONES_CCOEF` section in prmtop) and silently excluded those atoms. A second run attempt (job 41434642) crashed because the checkpoint had 40,330 particles but the updated OpenMM now saw 40,348.

**Fix:** Cleared all output files from replica_01, resubmitted test job (job 41454496). New run: System atoms 40,348, Production run completed. Both Ca273 and Ca274 present and coordinated. Triad geometry confirmed good in VMD.

**Key lesson:** Always verify `System atoms` in the run log matches prmtop NATOM after any new simulation setup.

---

## 2. VMD check scripts created/updated

- `vmd_check_ENG-S8_LPYPQPQLP.tcl` — updated path to `/home/mstefano/projects/md 1 ns replicas/ENG-S8_LPYPQPQLP/`, topology changed from prmtop to `minimized.pdb` (VMD 1.9.3 cannot read prmtop format)
- `vmd_check_MUT-S8_LPYPQPQLP.tcl` — new script, topology `minimized.pdb`, shows Asp42–His116–Ser288 triad, peptide resid 352–360, Ca361/362/363

Both scripts in `/home/mstefano/projects/marija_calculations/`.

Launch:
```bash
vmd -e /home/mstefano/projects/marija_calculations/vmd_check_ENG-S8_LPYPQPQLP.tcl
vmd -e /home/mstefano/projects/marija_calculations/vmd_check_MUT-S8_LPYPQPQLP.tcl
```

---

## 3. Local folder reorganisation

- Old standalone folders renamed → `/home/mstefano/projects/md 1 ns replicas/ENG-S8_LPYPQPQLP/` and `md 1 ns replicas/MUT-S8_LPYPQPQLP/`
- `/home/mstefano/projects/md 100 ns replicas/` created for 100 ns production runs

---

## 4. 100 ns production runs — all 6 systems set up and submitted

**Systems:**
| System | Atoms | Ca²⁺ | Peptide resids |
|--------|-------|------|----------------|
| ENG-S8_LPYPQPQLP | 40,348 | Ca273, Ca274 | 264–272 |
| ENG-S8_QLPYPQPQL | ~40k | Ca273, Ca274 | 264–272 |
| MUT-S8_LPYPQPQLP | 73,195 | Ca361, Ca362, Ca363 | 352–360 |
| MUT-S8_QLPYPQPQL | ~73k | Ca361, Ca362, Ca363 | 352–360 |
| WT-S8_LPYPQPQLP | ~73k | Ca361, Ca362, Ca363 | 352–360 |
| WT-S8_QLPYPQPQL | ~73k | Ca361, Ca362, Ca363 | 352–360 |

**Parameters (identical to 1 ns test):**
- ff14SB / TIP3P, 310 K, 2 fs timestep
- Ca²⁺: `frcmod.ions1lm_1264_tip3p` (12-6-4 LJ)
- Catalytic His: HID (forced, ND1 protonated facing Asp)
- NVT 0.1 ns (5→310 K) + NPT 0.5 ns (restraint 5→0 kcal/mol/Å²)
- Production: 100 ns, chunk_size 100 ns → production_01.dcd
- DCD every 100 ps → 1000 frames per replica
- 3 replicas per system (SLURM array), seeds: 112358132 / 471319856 / 839215768

**Input structures:** AF3 best models (seed/sample confirmed in prepare_systems.py):
- ENG-S8_LPYPQPQLP: seed5_sample3
- ENG-S8_QLPYPQPQL: seed3_sample0
- MUT-S8_LPYPQPQLP: seed3_sample2
- MUT-S8_QLPYPQPQL: seed2_sample1 (chain A extracted for propKa, full complex in prmtop)
- WT-S8_LPYPQPQLP: seed1_sample1
- WT-S8_QLPYPQPQL: seed4_sample3

**Submit scripts:** `/home/mstefano/projects/md 100 ns replicas/SYSTEM/submit_SYSTEM.sh`

**MN5 path:** `/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut/SYSTEM/`

**Status:** 18 jobs submitted (6 × array 1-3), pending as of end of session.

---

## 5. Next steps

- When 100 ns jobs complete: verify logs (`System atoms` correct for all 6), download results, run VMD checks
- After 100 ns analysis: submit 500 ns production runs using existing scripts in `marija_calculations/md_eng_wt_mut/` (re-upload those scripts to MN5, OpenMM will auto-resume from production_01.chk)
- Note: 100 ns submit scripts on MN5 have same name as 500 ns scripts — re-upload 500 ns versions before submitting
