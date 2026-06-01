# Session Summary — 2026-06-01

## 1. AlphaFold3 Predictions

### Jobs submitted and completed (MareNostrum5)
- **af3_mature_subtilsin_LPYPQPQLP_ca** — mature subtilisin E (263 aa) + LPYPQPQLP + Ca²⁺ ✅
- **af3_P04189_LPYPQPQLP_ca** — canonical subtilisin E P04189 (275 aa) + LPYPQPQLP + Ca²⁺ ✅
- **af3_subtilsin_pqpq_ca** — abandoned (PQPQ too short for AF3 MSA pipeline)

### Key results
- Best model: **seed-4_sample-0** for both systems
- **P1 residue = Q5** (Gln5 of LPYPQPQLP) — consistent with literature on subtilisin P1 preference
- Catalytic serine is **Ser209** in mature subtilisin (GTSMASP motif, subtilase family)
- Catalytic triad: **Asp20 – His52 – Ser209** (mature subtilisin) | **Asp32 – His64 – Ser221** (P04189)

| System | ipTM | Ranking score | OG–carbonyl C (Å) | P1 |
|--------|------|---------------|-------------------|----|
| Mature subtilisin | 0.69 | 0.75 | 3.71 | Q5 |
| P04189 | 0.64 | 0.72 | 3.13 | Q5 |

### Analysis script
- `marija_calculations/analyze_af3_distances.py` — distances + ipTM for all 25 models
- Output: `af3_mature_subtilsin_LPYPQPQLP_distances.xlsx`, `af3_P04189_LPYPQPQLP_distances.xlsx`

---

## 2. MD Simulation Setup

### System preparation
- Starting structure: AF3 best model (seed-4_sample-0, mature subtilisin)
- Protonation at **pH 8.5** with propKa 3.5.1 — all His → **HIE** (His52 pKa = 7.62 < 8.5)
- tleap: **ff14SB** force field, **TIP3P** water, cubic box 12 Å, neutralisation only
- System: ~11,725 water molecules, ~38,000 atoms total

### OpenMM simulation parameters (matching BSC glutenase pipeline)
| Parameter | Value |
|-----------|-------|
| Force field | ff14SB |
| Water | TIP3P |
| Temperature | 310 K |
| Time step | 2 fs |
| NVT equilibration | 0.1 ns (5 K → 310 K) |
| NPT equilibration | 0.5 ns (restraint 5 kcal/mol/Å², scaled to 0) |
| Production | 500 ns (5 × 100 ns chunks) |
| Replicas | 3 |
| Total simulation time | 1,500 ns |

### Files
| File | Location |
|------|----------|
| Prepared PDB | `marija_calculations/mature_subtilsin_ready.pdb` |
| AMBER topology | `marija_calculations/mature_subtilsin.prmtop` |
| AMBER coordinates | `marija_calculations/mature_subtilsin.inpcrd` |
| tleap script | `marija_calculations/tleap_mature_subtilsin.in` |
| SLURM script | `marija_calculations/md/submit_subtilsin_md.sh` |
| MN5 directory | `/gpfs/projects/bsc72/marija_stefanovic/subtilsin_md/` |

---

## 3. mfloor Glutenase MD Analysis

### Data source
`/gpfs/projects/bsc72/mfloor/glutenase/phase_a/ff14SB` (Martin Floor, BSC)

### Systems
| System | Replicas | Simulation time | Status |
|--------|----------|-----------------|--------|
| WT-S8 | 3 | ~480 ns each | ✅ Analysed |
| MUT-S8 | 3 | ~490 ns each | ✅ Analysed |
| REF-S9 | 3 | ~100 ns each | ⚠️ Different enzyme family, triad skipped |
| ENG-S8 | 3 | 500 ns each | ⚠️ Trajectories not stripped |

### Active site (WT-S8 / MUT-S8)
- Catalytic triad: **Asp42 – His116 – Ser288**
- P1 substrate residue: **Gln355** (Q5 of LPYPQPQLP)
- Structural Ca²⁺: resSeq 360, 361, 362

### Results summary

#### Catalytic triad distances
| Metric | WT-S8 | MUT-S8 |
|--------|-------|--------|
| Asp–His mean (Å) | 4.98 ± 0.49 | 5.03 ± 0.71 |
| His–Ser mean (Å) | 5.10 ± 0.98 | 5.75 ± 0.90 |
| Frames His–Ser < 3.5 Å | 8.8% | 0.6% |

#### Tetrahedral attack geometry
| Metric | WT-S8 | MUT-S8 |
|--------|-------|--------|
| Attack distance mean (Å) | 3.67 ± 0.59 | **3.29 ± 0.38** |
| BD angle mean (°) | 59.6 ± 25.0 | **79.3 ± 11.2** |
| Productive frames | 8.9% | **17.8%** |
| Best replica | rep1: 14.5% | rep3: **42.1%** |

#### Ca²⁺ coordination (mean coordination count)
| Ion | WT-S8 | MUT-S8 |
|-----|-------|--------|
| Ca360 | 6.8 ± 0.4 | 6.7 ± 0.5 |
| Ca361 | 7.6 ± 0.5 | 7.4 ± 0.5 |
| Ca362 | 3.2 ± 1.4 | 3.0 ± 1.1 |

**Key finding:** MUT-S8 shows superior attack geometry (shorter distance, better BD angle, 2× more productive frames) despite weaker His–Ser contact.

### Analysis pipeline
- Script: `marija_calculations/md_catalytic_analysis.py` (general, config-driven)
- Config: `marija_calculations/config_mfloor.json`
- Output: `marija_calculations/mfloor_analysis_results/`
  - `triad_geometry.xlsx` (29,050 frames)
  - `tetrahedral_attack.xlsx` (29,050 frames)
  - `ca_coordination.xlsx` (87,150 frames)
  - `plots/` — 6 publication-quality figures

---

## 4. Tools Installed / Set Up

| Tool | Environment | Purpose |
|------|-------------|---------|
| gemmi | prep | CIF → PDB conversion |
| propKa 3.5.1 | tools | Protonation state prediction |
| AmberTools (tleap) | prep | System preparation |
| OpenMM | MN5 conda env | MD engine |
| mdtraj 1.11 | tools | Trajectory analysis |
| md_enzyme_analysis | tools | Catalytic geometry analysis |
| VMD 1.9.3 | tools | Trajectory visualization |
| seaborn / matplotlib | tools | Plotting |
| python-docx | tools | Report generation |

---

## 5. GitHub Pushes

Repository: `git@github.com:marijastefanovic-collab/calculations.git`

| Commit | File |
|--------|------|
| `f66bbd9` | `md_setup_report.md` — MD simulation setup documentation |
| `69d9819` | `mfloor_md_results_report.md` — mfloor analysis results report |

---

## 6. Pending / Next Steps

- [ ] Wait for subtilisin MD job to complete on MN5 (3 replicas × 500 ns, ~5 days)
- [ ] Download subtilisin trajectories and run analysis with `config_subtilsin.json`
- [ ] Complete REF-S9 analysis once correct triad residue numbers are confirmed
- [ ] Analyse ENG-S8 once trajectories are water-stripped on MN5
- [ ] REF-S9 simulation still incomplete on MN5 (200/500 ns)
