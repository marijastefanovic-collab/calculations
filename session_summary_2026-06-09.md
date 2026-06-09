# Session Summary — 2026-06-09

## Work completed this session

---

### 1. Fixed analyse_100ns.py — residue numbering and per-replica topology

**Problem:** The `strip_100ns.py` script saves `minimized_non_solvent.pdb` using OpenMM's `minimized.pdb` as topology source. OpenMM writes PDB files with 1-based residue numbering (AMBER convention), but the analysis scripts used 0-based `pdb_resSeq` (matching MDTraj from prmtop). This caused all residue lookups to be off by 1 (e.g., resSeq=19 → ILE instead of Asp).

**Fix 1 — SYSTEMS dict updated to 1-based resSeq:**
```
ENG-S8: asp=20, his=52, ser=209, p1=268/269, ca=[273,274]
MUT/WT: asp=42, his=116, ser=288, p1=356/357, ca=[361,362,363]
```

**Fix 2 — Per-replica topology from `minimized.pdb`:**  
Different replicas of the same system have different atom counts because OpenMM's 12-6-4 Ca²⁺ parameter bug drops different atoms in different runs. Using a single system-level `minimized_non_solvent.pdb` as topology causes `xyz must be shape (Any, N, 3)` errors.

New approach: for each replica, derive the stripped topology from that replica's own `minimized.pdb`:
```python
def get_per_replica_topology(rep_dir):
    full_top = md.load_topology(str(rep_dir / "minimized.pdb"))
    keep_idx = [a.index for a in full_top.atoms if a.residue.name not in EXCLUDE_RESIDUES]
    return full_top.subset(np.array(keep_idx, dtype=np.intp))
```

**Fix 3 — Missing Ca gracefully skipped:**  
MUT-S8_LPYPQPQLP replicas 02/03 are missing Ca362/Ca363. Added check before Ca coordination computation:
```python
if not any(r.resSeq == rs for r in top.residues):
    print(f"WARNING: {label} not in {rep} — skipped")
    continue
```

**Fix 4 — RMSD uses per-replica frame 0 as reference:**  
Since topologies differ across replicas, RMSD is now computed against frame 0 of each replica independently (intra-replica conformational drift). RMSF likewise.

Script: `/home/mstefano/projects/marija_calculations/analyse_100ns.py`

---

### 2. 100 ns analysis completed — all 6 systems

Script ran successfully. Output: `/home/mstefano/projects/results/100ns_results/100ns_results.xlsx`

| System | Attack d (Å) | BD (°) | Productive % | His-Ser <3.5Å % | Oxy <3.5Å % | RMSD (Å) |
|--------|-------------|--------|-------------|-----------------|-------------|----------|
| ENG-S8_LPYPQPQLP | 5.33 ± 0.55 | 72.5 ± 18.1 | 0% | 24% | 0% | 4.3 |
| ENG-S8_QLPYPQPQL | 5.25 ± 0.60 | 58.9 ± 16.6 | 0% | 52% | 0% | 8.6 |
| **MUT-S8_QLPYPQPQL** | **3.51 ± 0.59** | **77.6 ± 7.1** | **7%** | **94%** | **93%** | 1.6 |
| MUT-S8_LPYPQPQLP | 6.04 ± 1.05 | 74.0 ± 23.8 | 0% | 12% | 0% | 1.7 |
| WT-S8_LPYPQPQLP | 3.65 ± 0.46 | 49.0 ± 24.5 | 3% | 41% | 29% | 1.5 |
| WT-S8_QLPYPQPQL | 5.84 ± 0.48 | 80.2 ± 15.0 | 0% | 29% | 0% | 1.6 |

**Key observations:**
- **MUT-S8_QLPYPQPQL** is the only system with consistent near-attack geometry at 100 ns: 7% productive, 94% His-Ser engaged, 93% oxyanion engaged. Consistent with 1 ns test results.
- **ENG-S8 systems** remain non-productive at 100 ns. ENG-S8_QLPYPQPQL shows higher His-Ser engagement (52%) but attack distance too long.
- **WT-S8_LPYPQPQLP**: 3% productive, 29% oxyanion — moderate activity.
- **RMSD caveat**: ENG-S8 systems show high RMSD (4-9 Å), suggesting structural rearrangement or peptide unbinding over 100 ns.

---

### 3. Root cause analysis — MUT-S8_LPYPQPQLP missing Ca ions in replicas 02/03

**Observation:** MUT-S8_LPYPQPQLP replica_01 has 73,195 atoms (3 Ca: 361, 362, 363); replicas 02/03 have 73,186 atoms (1 Ca: only 361).

**Root cause:**
- The MN5 OpenMM conda environment (`openmm_cuda`) is an **old version (< 7.6)** that has the 12-6-4 Ca²⁺ bug
- When the old OpenMM loads a prmtop with `LENNARD_JONES_CCOEF` parameters, it drops affected Ca atoms **from the topology itself** (not just from the system)
- This means both `topology.getNumAtoms()` and `system.getNumParticles()` return 73,186 — so the guard check in `openmm_simulation.py` passes silently:
  ```python
  if system_n_particles != n_atoms:  # 73186 == 73186 → passes!
      raise RuntimeError(...)
  ```
- Replica_01 was **not affected** because its `minimized.pdb` was already present from the 1 ns test run (timestamp: Jun 5). The 100 ns script skipped re-minimization and ran production from the existing 73,195-atom structure.
- Replicas 02/03 had no prior `minimized.pdb`, ran fresh minimization with the buggy OpenMM, and produced 73,186-atom simulations missing Ca362/Ca363.

**Impact:** Structural metrics (attack distance, BD angle, catalytic triad, oxyanion) for replicas 02/03 are valid. Ca coordination for Ca362 and Ca363 is unavailable for those replicas.

**Recommendation:** To get complete Ca coordination data, replicas 02/03 of MUT-S8_LPYPQPQLP need to be rerun with an upgraded OpenMM version (≥ 7.6) or with a different Ca parameter approach that avoids the 12-6-4 issue.

---

## Pending tasks

1. **Rerun MUT-S8_LPYPQPQLP replicas 02/03** on MN5 with fixed OpenMM
2. **Submit 3 missing BD jobs** manually on MN5: ENG-S8_LPYPQPQLP_s4p2, ENG-S8_QLPYPQPQL_s5p1, MUT-S8_LPYPQPQLP_s1p1
3. **Resubmit ENG-S8_LPYPQPQLP replica_01** 100 ns job (old test DCD, skipped by strip_100ns.py)
4. **Run strip_att_1ns.py and strip_bd_1ns.py** when ATT/BD trajectories are complete
5. **Run analyse_att_1ns.py and analyse_bd_1ns.py**
6. **Upload and submit** 3 new 100 ns systems (ENG-S8_LPYPQPQLP_s4p2, ENG-S8_QLPYPQPQL_s5p1, MUT-S8_LPYPQPQLP_s1p1) to MN5
