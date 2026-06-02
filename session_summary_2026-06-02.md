# Session Summary — 2026-06-02

## 1. ENG-S8 Trajectory Analysis

### Trajectories downloaded and stripped
- Raw trajectories copied from MN5 `~/glutenase_ff14SB` → local `~/glutenase_ff14SB/ENG-S8/`
- Stripped using `createNonSolventTrajectories.py` (`md_enzyme_analysis`), variant `non_solvent`, kept Ca²⁺, excluded Na/Cl/K/Mg
- Output: 15 stripped DCDs (5 chunks × 3 replicas) + `minimized_non_solvent.pdb` per replica

### ENG-S8 active site
- Catalytic triad: **Asp20 – His52 – Ser209**
- P1 substrate residue: **Gln268** (Q5 of LPYPQPQLP)
- Structural Ca²⁺: resSeq 273, 274 (only 2 ions, vs 3 in WT/MUT)

---

## 2. Corrected Triad Analysis (all systems)

### MDTraj ND1/NE2 label swap — critical finding
MDTraj relabels HID as HIE when saving stripped PDB files, **swapping ND1 and NE2 atom names**. VMD inspection of original input PDB confirmed canonical orientation: **ND1→Asp, NE2→Ser**. All previous triad distances using default atom names were therefore wrong.

Corrected analysis used `compute_triad_geometry` from `md_enzyme_analysis.catalytic` with `his_acceptor="NE2"`, `his_donor="ND1"` to compensate for the swap.

### Corrected results (all replicas combined)

#### Catalytic triad distances
| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Asp–His [ND1] mean (Å) | 3.34 ± 1.03 | 4.07 ± 2.06 | 3.87 ± 1.57 |
| Asp–His < 3.5 Å | **75.9%** | 71.9% | 62.9% |
| His–Ser [NE2] mean (Å) | 5.25 ± 0.96 | 5.62 ± 1.57 | 5.69 ± 1.29 |
| His–Ser < 3.5 Å | 5.2% | **14.7%** | 0.0% |

#### Tetrahedral attack geometry (unchanged from 2026-06-01)
| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Attack distance mean (Å) | 3.67 ± 0.59 | **3.29 ± 0.38** | 5.06 ± 0.76 |
| BD angle mean (°) | 59.6 ± 25.0 | **79.3 ± 11.2** | 46.7 ± 23.4 |
| Productive frames | 8.9% | **17.8%** | 0.2% |
| Best replica | rep1: 14.5% | rep3: 42.1% | rep1: 0.6% |

#### Ca²⁺ coordination
| Ion | WT-S8 | MUT-S8 | ENG-S8 |
|-----|-------|--------|--------|
| Ca360/Ca273 | 6.8 ± 0.4 | 6.7 ± 0.5 | 3.0 ± 0.1 |
| Ca361/Ca274 | 7.6 ± 0.5 | 7.4 ± 0.5 | 6.3 ± 0.5 |
| Ca362 | 3.2 ± 1.4 | 3.0 ± 1.1 | — |

---

## 3. His Protonation State Analysis

### Setup
- All systems: **HID tautomer** (HD1 on ND1, NE2 free), pH 7.0 (Martin Floor)
- Confirmed by prmtop atom inventory and VMD inspection of original PDB

### propKa (protein-only, pH 7.0)
- ENG-S8: Asp20 pKa 4.53 (deprotonated), His52 pKa 5.65 (neutral HID) ✓
- WT-S8: Asp42 pKa 4.84 (deprotonated), His116 pKa 5.51 (neutral HID) ✓
- **MUT-S8: Asp42 pKa 8.03 (predicted protonated at pH 7.0!)** — mutations increase burial
- Note: propKa severely underestimates buried His pKa; 5.5–5.6 values are unreliable; conclusion is His is neutral

### Key finding
ND1 is within H-bond distance of Asp(OD) in 63–76% of frames (mean 3.34–4.07 Å). His–Ser (NE2) contact is weak in all systems (0–14.7%), consistent with a pre-catalytic Michaelis complex.

---

## 4. Files

| File | Location |
|------|----------|
| Stripped ENG-S8 DCDs | `~/glutenase_ff14SB/ENG-S8/replica_0X/production_*_non_solvent.dcd` |
| ENG-S8 stripped topology | `~/glutenase_ff14SB/ENG-S8/replica_01/minimized_non_solvent.pdb` |
| Triad results | `marija_calculations/mfloor_analysis_results/triad_geometry.xlsx` |
| Attack results | `marija_calculations/mfloor_analysis_results/tetrahedral_attack.xlsx` |
| Ca coordination | `marija_calculations/mfloor_analysis_results/ca_coordination.xlsx` |
| His protonation | `marija_calculations/mfloor_analysis_results/his_protonation_analysis.xlsx` |
| propKa output | `~/glutenase_ff14SB/parameters/ambertools/*_protonly.pka` |
| Report (md) | `marija_calculations/mfloor_md_results_report.md` |
| Report (docx) | `marija_calculations/mfloor_md_results_report.docx` |

---

## 5. Pending / Next Steps

- [ ] Wait for subtilisin MD job to complete on MN5 (3 replicas × 500 ns, ~5 days from 2026-06-01)
- [ ] Download subtilisin trajectories and run analysis with `config_subtilsin.json`
- [ ] Complete REF-S9 analysis once correct triad residue numbers are confirmed
- [ ] Consider re-running MUT-S8 simulation with Asp42 protonated (pKa 8.03)
- [ ] REF-S9 simulation still incomplete on MN5 (200/500 ns)
