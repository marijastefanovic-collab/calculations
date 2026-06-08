# Session Summary — 2026-06-08

## Work completed this session

### 1. AF3 complex_predictions_2.6 — model selection

Four models selected from the new AF3 2.6 batch for 1 ns test MD runs:

| # | System | AF3 source | Peptide |
|---|--------|-----------|---------|
| 1 | ENG-S8_LPYPQPQLP_s4p2 | seed-4, sample-2 | LPYPQPQLP |
| 2 | ENG-S8_LPYPQPQLP_s5p4 | seed-5, sample-4 | LPYPQPQLP |
| 3 | ENG-S8_QLPYPQPQL_s5p1 | seed-5, sample-1 | QLPYPQPQL |
| 4 | MUT-S8_LPYPQPQLP_s1p1 | seed-1, sample-1 | LPYPQPQLP |

Source CIFs: `/home/mstefano/projects/AF3 models/complex_predictions_2.6/<system>/output/.../model.cif`

### 2. MD system preparation

Wrote preparation script: `/home/mstefano/projects/md 1 ns replicas/prepare_new_2p6.py`

Pipeline per system:
1. CIF → PDB via gemmi (renamed Ca chains CAA/CAB/CAC → single-char C/D/E)
2. propKa 3.5.1 at pH 8.5 for His tautomer assignment
3. `prepare_pdb()`: catalytic His → HID (forced), others → HIP/HIE by pKa; all Ca²⁺ consolidated to chain C with unique sequential resids
4. tleap input written (ff14SB + TIP3P + `frcmod.ions1lm_1264_tip3p`, 12 Å box)
5. SLURM submit script (1 ns, replica_01 only, seed 112358132, acc_bscls, 2h)

**His protonation (pH 8.5):**
- ENG-S8 systems: His52=HID (catalytic, forced); His5,27,55,214,226=HIE
- MUT-S8 system: His116=HID (catalytic, forced); His48,119,293=HIE

**Ca²⁺ handling:**
- ENG-S8 (2 Ca): Ca C 273, Ca C 274 in ready PDB
- MUT-S8 (3 Ca): Ca C 360, Ca C 361, Ca C 362 in ready PDB
- All Ca consolidated to chain C (single chain, unique resids) — prevents tleap "new molecule" issue

### 3. tleap — all 4 systems passed

Run locally with `/home/mstefano/micromamba/envs/tools/bin/tleap`.

| System | NATOM | Ca²⁺ (solvated resids) | tleap result |
|--------|-------|------------------------|-------------|
| ENG-S8_LPYPQPQLP_s4p2 | 39,913 | Ca273, Ca274 | Errors = 0 ✓ |
| ENG-S8_LPYPQPQLP_s5p4 | 40,366 | Ca273, Ca274 | Errors = 0 ✓ |
| ENG-S8_QLPYPQPQL_s5p1 | 39,625 | Ca273, Ca274 | Errors = 0 ✓ |
| MUT-S8_LPYPQPQLP_s1p1 | 73,918 | Ca361, Ca362, Ca363 | Errors = 0 ✓ |

prmtop/inpcrd copied to each `replica_01/input_files/`.

### 4. Upload to MN5

All 4 system directories uploaded via rsync to:
`/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut/<system>/`

Each directory contains: `*_ready.pdb`, `tleap_*.in`, `leap_*.log`, `*_solvated.pdb`, `*`.prmtop/inpcrd`, `replica_01/input_files/`, `scripts/openmm_simulation.py`, `submit_*_test.sh`.

## Next steps

Submit 1 ns jobs on MN5 when pending jobs free up queue slots:

```bash
MN5_BASE=/gpfs/projects/bsc72/marija_stefanovic/md_eng_wt_mut
cd $MN5_BASE/ENG-S8_LPYPQPQLP_s4p2 && sbatch submit_ENG-S8_LPYPQPQLP_s4p2_test.sh
cd $MN5_BASE/ENG-S8_LPYPQPQLP_s5p4 && sbatch submit_ENG-S8_LPYPQPQLP_s5p4_test.sh
cd $MN5_BASE/ENG-S8_QLPYPQPQL_s5p1  && sbatch submit_ENG-S8_QLPYPQPQL_s5p1_test.sh
cd $MN5_BASE/MUT-S8_LPYPQPQLP_s1p1  && sbatch submit_MUT-S8_LPYPQPQLP_s1p1_test.sh
```

After completion:
- Verify `System atoms` in SLURM log matches NATOM above (OpenMM 12-6-4 Ca²⁺ atom-drop check)
- Inspect trajectories in VMD: Ca²⁺ coordination, triad geometry
- If geometry is productive → submit 100 ns 3-replica runs
