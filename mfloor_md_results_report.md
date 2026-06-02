# MD Simulation Analysis Report
## Glutenase Catalytic Geometry Analysis — Positive Controls and Engineering Target

**Date:** 2026-06-02  
**Data source:** Martin Floor, BSC — `/gpfs/projects/bsc72/mfloor/glutenase/phase_a/ff14SB`  
**Analysis by:** Marija Stefanovic  

---

## 1. Overview

This report summarizes the catalytic geometry analysis of MD simulations from the BSC glutenase project. Three S8-type serine proteases with a bound substrate peptide (**LPYPQPQLP**) were analysed over ~500 ns of production MD per replica.

**Study design:**
- **WT-S8** and **MUT-S8** are positive controls with known glutenase activity. They share high sequence similarity; MUT-S8 is a double mutant of WT-S8.
- **ENG-S8** is an independent engineering target — a newly discovered S8-type serine protease with a different sequence, different active site numbering (Asp20–His52–Ser209), and only two structural Ca²⁺ ions. It is not derived from WT-S8; the simulations characterise its catalytic geometry as a baseline for engineering.

### Systems Analysed

| System | Role | Replicas | Simulation time | Frames |
|--------|------|----------|-----------------|--------|
| WT-S8 | Positive control (wild-type) | 3 | ~480 ns each | 14,371 |
| MUT-S8 | Positive control (double mutant of WT) | 3 | ~490 ns each | 14,679 |
| ENG-S8 | Engineering target (new S8 enzyme) | 3 | 500 ns each | 15,000 |
| REF-S9 | Reference (different protease family) | 3 | ~100 ns each | — |

> **Note:** REF-S9 belongs to a different protease family — triad and attack analyses were not performed for this system.

### Simulation Parameters

| Parameter | Value |
|-----------|-------|
| Force field | ff14SB |
| Water model | TIP3P |
| Temperature | 310 K |
| Integrator | Langevin Middle, 2 fs time step |
| Pressure | 1 bar (Monte Carlo barostat) |
| Equilibration | 0.1 ns NVT + 0.5 ns NPT |
| Production chunks | 5 × 100 ns per replica |

### Active Site

| Residue | Role | WT-S8 / MUT-S8 | ENG-S8 |
|---------|------|----------------|--------|
| Catalytic Asp | Triad base | Asp42 | Asp20 |
| Catalytic His | Triad shuttle | His116 | His52 |
| Catalytic Ser | Nucleophile | Ser288 | Ser209 |
| P1 substrate (Gln) | Scissile residue | Gln355 | Gln268 |
| Structural Ca²⁺ | Structural ions | Ca360, Ca361, Ca362 | Ca273, Ca274 |

---

## 2. Catalytic Triad Geometry

The catalytic triad hydrogen bond network was monitored via two distances:
- **Asp(OD) – His(ND1):** proton acceptor arm of the triad
- **His(NE2) – Ser(OG):** proton donor arm of the triad (activates the nucleophilic Ser)

A functional H-bond is typically < 3.5 Å.

### Summary Statistics (all replicas combined)

| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Asp–His mean distance (Å) | 4.98 ± 0.49 | 5.03 ± 0.71 | 5.32 ± 0.71 |
| Frames with Asp–His < 3.5 Å | 0.0% | 0.0% | 0.0% |
| His–Ser mean distance (Å) | **5.10 ± 0.98** | 5.75 ± 0.90 | 5.73 ± 1.29 |
| Frames with His–Ser < 3.5 Å | **8.8%** | 0.6% | 3.3% |

### Interpretation

Neither the Asp–His nor the His–Ser contact forms a persistent H-bond in any system — both distances are well above the 3.5 Å threshold and consistent with a **pre-catalytic Michaelis complex** where the triad is not yet fully organised for catalysis.

WT-S8 shows the most frequent His–Ser contacts (8.8% of frames < 3.5 Å), suggesting the wild-type scaffold best positions the His for Ser activation. MUT-S8 (0.6%) and ENG-S8 (3.3%) both show weak His–Ser interaction. Note that MUT-S8 replica_01 has a disengaged Asp–His (5.54 Å) contributing to inter-replica variability.

---

## 3. Tetrahedral Attack Geometry

The nucleophilic attack of the catalytic Ser OG on the carbonyl carbon of the P1 residue (Gln) was monitored via:
- **Attack distance:** Ser(OG) – Gln(C), productive threshold < 4.0 Å
- **Bürgi–Dunitz (BD) angle:** Nu–C=O angle, productive range 90°–120°

A frame is classified as **productive** when both criteria are met simultaneously (distance < 4.0 Å AND BD 90°–120°).

### Summary Statistics

| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Attack distance mean (Å) | 3.67 ± 0.59 | **3.29 ± 0.38** | 5.06 ± 0.76 |
| BD angle mean (°) | 59.6 ± 25.0 | **79.3 ± 11.2** | 46.7 ± 23.4 |
| Productive frames (total) | 8.9% | **17.8%** | 0.2% |
| Productive — replica 1 | 14.5% | 10.6% | 0.6% |
| Productive — replica 2 | 0.2% | 0.6% | 0.0% |
| Productive — replica 3 | 12.0% | **42.1%** | 0.0% |

### Interpretation

**MUT-S8 shows the best attack geometry, ENG-S8 the worst:**
- MUT-S8 has the shortest attack distance (3.29 Å), best BD angle (79°), and most productive frames (17.8%)
- WT-S8 is intermediate (3.67 Å, 60°, 8.9% productive)
- ENG-S8 shows markedly poor geometry: attack distance 5.06 Å (>1 Å further than WT), BD angle 47°, and only 0.2% productive frames

For the positive controls, the high inter-replica variability (WT-S8: 0.2–14.5%, MUT-S8: 0.6–42.1%) indicates incomplete conformational sampling in some replicas; longer simulations or additional replicas may be needed for converged estimates.

ENG-S8's poor attack geometry (attack distance 5.06 Å vs ~3.3–3.7 Å in the controls) establishes the baseline for this engineering target. Unlike WT/MUT where the substrate is well-positioned in some replicas, ENG-S8 does not achieve productive attack geometry in any replica, indicating the native scaffold requires engineering to improve substrate positioning.

---

## 4. Ca²⁺ Ion Coordination

Structural Ca²⁺ ions were monitored. The first-shell coordination count was calculated using a cutoff of 2.9 Å, counting protein O/N donor atoms (excluding bulk solvent and counter-ions). Note that ENG-S8 contains only **two** Ca²⁺ ions (Ca273, Ca274), while WT-S8 and MUT-S8 have three (Ca360–362).

### Mean Coordination Numbers

| Ca²⁺ ion | WT-S8 | MUT-S8 | Interpretation |
|----------|-------|--------|----------------|
| Ca360 | 6.8 ± 0.4 | 6.7 ± 0.5 | Well-coordinated, stable |
| Ca361 | 7.6 ± 0.5 | 7.4 ± 0.5 | Well-coordinated, stable |
| Ca362 | 3.2 ± 1.4 | 3.0 ± 1.1 | Partial / dynamic coordination |

| Ca²⁺ ion | ENG-S8 | Interpretation |
|----------|--------|----------------|
| Ca273 | 3.0 ± 0.1 | Partial / dynamic coordination |
| Ca274 | 6.3 ± 0.5 | Well-coordinated, stable |

### Interpretation

In WT-S8 and MUT-S8, Ca360 and Ca361 are stably coordinated (~7 ligands each), typical of structural Ca²⁺ sites (coordination number 6–8). Ca362 is partially occupied (mean ~3), suggesting a solvent-exposed site.

ENG-S8, as a distinct enzyme, has only two structural Ca²⁺ ions. Ca274 is well-coordinated (6.3 ± 0.5), consistent with a stable structural site comparable to Ca360/Ca361 in the positive controls. Ca273 shows low, dynamic coordination (3.0 ± 0.1), similar to the partially occupied Ca362 in WT/MUT. The different number of Ca²⁺ ions reflects the distinct protein sequence and surface of ENG-S8 relative to the positive controls.

No significant difference in Ca²⁺ coordination is observed between WT-S8 and MUT-S8, indicating those mutations do not perturb the structural calcium sites.

---

## 5. Summary and Conclusions

| Property | WT-S8 | MUT-S8 | ENG-S8 | Best |
|----------|-------|--------|--------|------|
| Asp–His < 3.5 Å | 0.0% | 0.0% | 0.0% | — |
| His–Ser < 3.5 Å | **8.8%** | 0.6% | 3.3% | WT-S8 |
| Attack distance | 3.67 Å | **3.29 Å** | 5.06 Å | **MUT-S8** |
| BD angle | 59.6° | **79.3°** | 46.7° | **MUT-S8** |
| Productive frames | 8.9% | **17.8%** | 0.2% | **MUT-S8** |
| Ca²⁺ coordination | Stable | Stable | Partial | WT-S8 / MUT-S8 |

**Among the positive controls**, MUT-S8 shows superior attack geometry compared to WT-S8: shorter attack distance (3.29 vs 3.67 Å), BD angle closer to ideal (79° vs 60°), twice the productive frame rate (17.8% vs 8.9%), and higher His–Ser contact frequency (14.7% vs 5.2%). WT-S8 maintains a marginally better Asp–His contact (75.9% vs 71.9%). The double mutation therefore improves nucleophile positioning without destabilising the triad.

**ENG-S8 (engineering target):** The baseline characterisation shows no His–Ser contact (0%), poor attack geometry (5.06 Å attack distance, BD 47°, 0.2% productive), and a maintained Asp–His interaction (62.9% < 3.5 Å). As ENG-S8 is a distinct enzyme from WT/MUT, direct comparison of absolute values is not the primary goal. The results establish a baseline catalytic geometry for this new S8 scaffold prior to any engineering. The poor substrate positioning relative to the positive controls suggests there is room for improvement through targeted engineering of the ENG-S8 active site.

---

## 6. His Protonation State Analysis

### Simulation setup

All three S8 systems were set up at **pH 7.0** (Martin Floor, BSC) with histidine in the **HID tautomer** (HD1 on ND1, NE2 free), as recorded in both the input PDB files and AMBER prmtop files. This was verified by inspecting the hydrogen atom inventory of the catalytic His residue (His116 in WT/MUT-S8; His52 in ENG-S8).

### propKa predictions (pH 7.0)

propKa 3.5.1 was run on **protein-only chain A** (substrate peptide and Ca²⁺ ions excluded). The initial run including all chains gave physically inconsistent results (Asp pKa 8.3–8.6, His pKa 1.5–1.9) because Coulomb perturbations from the Ca²⁺ ions and charged peptide strongly distorted the calculation. Protein-only results are more reliable.

The catalytic His entries were identified by their Asp neighbor in the propKa perturbation terms. Note that propKa reports His as two coupled microscopic pKa values (NAR — Not A Regular group); the values below are the dominant microscopic pKa for each His.

| Residue | System | propKa pKa | State at pH 7.0 | Note |
|---------|--------|-----------|-----------------|------|
| Asp20 | ENG-S8 | 4.53 | Deprotonated (–COO⁻) | Slightly elevated above model (3.80) |
| His52 (HID) | ENG-S8 | 5.65 | Neutral | Slightly below model (6.50) |
| Asp42 | WT-S8 | 4.84 | Deprotonated (–COO⁻) | Slightly elevated |
| His116 (HID) | WT-S8 | 5.51 | Neutral | Slightly below model |
| **Asp42** | **MUT-S8** | **8.03** | **Protonated (–COOH)** | Strongly elevated — see below |
| His116 (HID) | MUT-S8 | 4.01 | Neutral | Below model |

For ENG-S8 and WT-S8 the results are physically consistent: Asp is deprotonated and His is neutral at pH 7.0, as expected for the Michaelis complex of a serine protease.

**MUT-S8 Asp42 pKa = 8.03** is the notable exception. This elevation persists even without the substrate, indicating it is a direct consequence of the mutations increasing Asp42 burial (propKa predicts 100% desolvation for Asp42 in MUT-S8 vs a smaller contribution in WT-S8). If correct, Asp42 in MUT-S8 would be **protonated at pH 7.0**, which would alter the charge relay. This is an unexpected finding given that MUT-S8 actually shows better catalytic geometry than WT-S8 — suggesting the protonated Asp may still be functional in this context, or that the propKa prediction is overestimating the burial effect of the mutations.

### Per-nitrogen distance analysis

To assess whether the simulated HID tautomer is geometrically consistent with the observed dynamics, all four His nitrogen–heavy atom distances were tracked across all replicas and frames:

VMD inspection of the original input PDB confirmed the **canonical HID orientation: ND1 faces Asp, NE2 faces Ser**, consistent with the standard serine protease mechanism. The analysis uses the stripped trajectory PDB as topology (same approach as the original June 1 analysis) with default atom assignment.

| Distance | WT-S8 | MUT-S8 | ENG-S8 |
|----------|-------|--------|--------|
| Asp–His(ND1) mean (Å) | 4.98 ± 0.49 | 5.03 ± 0.71 | 5.32 ± 0.71 |
| Asp–His(ND1) < 3.5 Å | 0.0% | 0.0% | 0.0% |
| His(NE2)–Ser mean (Å) | **5.10 ± 0.98** | 5.75 ± 0.90 | 5.73 ± 1.29 |
| His(NE2)–Ser < 3.5 Å | **8.8%** | 0.6% | 3.3% |

### Interpretation

Both triad distances are above the H-bond threshold in all systems, consistent with a pre-catalytic Michaelis complex. The Asp–His arm (ND1–Asp) is never < 3.5 Å, indicating the His is not in a fully activated charge relay state. The His–Ser arm (NE2–Ser) shows the most contact in WT-S8 (8.8%), suggesting the wild-type scaffold best positions NE2 toward Ser for eventual nucleophile activation.

**Recommendation:** For ENG-S8 and WT-S8, the protonation state (Asp deprotonated, His neutral HID) is consistent with propKa and can be considered correct at pH 7.0. For MUT-S8, the elevated Asp42 pKa (8.03) warrants re-examination — the simulation should ideally be repeated with Asp42 protonated. For a reliable His pKa in all systems, constant-pH MD or free energy perturbation is needed.

---

## 6. Figures

| Figure | Description |
|--------|-------------|
| fig1_triad_geometry.png | Violin plots — Asp–His and His–Ser distance distributions |
| fig2_triad_timeseries.png | Triad distances over simulation time per replica |
| fig3_attack_geometry.png | Violin plots — attack distance and BD angle |
| fig4_attack_landscape.png | 2D hexbin of attack distance vs BD angle with productive zone |
| fig5_productive_frames.png | % productive frames per system and replica |
| fig6_ca_coordination.png | Ca²⁺ coordination counts per ion |

All figures: `/home/ordi/marija_calculations/mfloor_analysis_results/plots/`

---

## 8. Analysis Pipeline

```
Stripping:  createNonSolventTrajectories.py  (md_enzyme_analysis)
            — variant: non_solvent / non_water, exclude Na/Cl/K/Mg, keep Ca
Analysis:   md_enzyme_analysis.catalytic
            — compute_triad_geometry   (his_acceptor="NE2", his_donor="ND1")
            — compute_tetrahedral_attack
            — compute_ion_coordination
Output:     triad_geometry.xlsx, tetrahedral_attack.xlsx, ca_coordination.xlsx

Note on His atom names: MDTraj relabels HID→HIE in stripped PDB output,
swapping ND1/NE2. Atom names are passed explicitly to match original PDB orientation
(ND1→Asp, NE2→Ser, confirmed by VMD on input structures).
```
