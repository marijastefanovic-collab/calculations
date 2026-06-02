# MD Simulation Analysis Report
## Glutenase Variants — Catalytic Geometry Analysis

**Date:** 2026-06-02  
**Data source:** Martin Floor, BSC — `/gpfs/projects/bsc72/mfloor/glutenase/phase_a/ff14SB`  
**Analysis by:** Marija Stefanovic  

---

## 1. Overview

This report summarizes the catalytic geometry analysis of glutenase MD simulations from the BSC glutenase project. Three S8-family enzyme variants with a bound substrate peptide (**LPYPQPQLP**) were analysed over 500 ns of production MD per replica.

### Systems Analysed

| System | Description | Replicas | Simulation time | Frames |
|--------|-------------|----------|-----------------|--------|
| WT-S8 | Wild-type glutenase + LPYPQPQLP | 3 | ~480 ns each | 14,371 |
| MUT-S8 | Mutant glutenase + LPYPQPQLP | 3 | ~490 ns each | 14,679 |
| ENG-S8 | Engineered variant + LPYPQPQLP | 3 | 500 ns each | 15,000 |
| REF-S9 | Reference enzyme (different family) | 3 | ~100 ns each | — |

> **Note:** REF-S9 belongs to a different protease family with a distinct active site numbering — catalytic triad and attack geometry analyses were not performed for this system.

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

Distances computed with correct His atom assignment: **Asp–His = His(ND1) – Asp(OD)** and **His–Ser = His(NE2) – Ser(OG)**, consistent with the HID tautomer and ND1→Asp, NE2→Ser orientation confirmed by VMD.

| Metric | WT-S8 | MUT-S8 | ENG-S8 |
|--------|-------|--------|--------|
| Asp–His mean distance (Å) | **3.34 ± 1.03** | 4.07 ± 2.06 | 3.87 ± 1.57 |
| Frames with Asp–His < 3.5 Å | **75.9%** | 71.9% | 62.9% |
| His–Ser mean distance (Å) | 5.25 ± 0.96 | 5.62 ± 1.57 | 5.69 ± 1.29 |
| Frames with His–Ser < 3.5 Å | 5.2% | **14.7%** | 0.0% |

### Interpretation

The **Asp–His interaction is well-maintained** in all three systems: ND1 is within H-bond distance of Asp(OD) in 63–76% of frames, consistent with the canonical HID orientation and a stable Asp–His interaction in the Michaelis complex.

The **His–Ser arm is weak** across all systems (mean 5.2–5.7 Å), indicating NE2 rarely reaches the Ser nucleophile. MUT-S8 shows the most frequent His–Ser contacts (14.7%), driven entirely by replica_02 (44%). WT-S8 is intermediate (5.2%). ENG-S8 shows no His–Ser contact (0%), consistent with its poor catalytic geometry overall.

Note that MUT-S8 replica_01 shows a disengaged Asp–His (6.53 Å mean), explaining the high inter-replica variability in that system.

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

ENG-S8's poor attack geometry suggests that the engineering altered the active site architecture in a way that disrupts substrate positioning for catalysis. The high inter-replica variability in WT-S8 (0.2–14.5%) and MUT-S8 (0.6–42.1%) indicates incomplete conformational sampling in some replicas.

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

ENG-S8 has only two Ca²⁺ ions. Ca274 is well-coordinated (6.3), comparable to Ca360/Ca361 in the other variants. Ca273 shows low coordination (3.0 ± 0.1), similar to Ca362 in WT/MUT. The absence of a third Ca²⁺ in ENG-S8 may reflect differences in the engineered protein surface or crystallisation conditions.

No significant difference in Ca²⁺ coordination is observed between WT-S8 and MUT-S8, indicating those mutations do not perturb the structural calcium sites.

---

## 5. Summary and Conclusions

| Property | WT-S8 | MUT-S8 | ENG-S8 | Best |
|----------|-------|--------|--------|------|
| Asp–His < 3.5 Å | **75.9%** | 71.9% | 62.9% | WT-S8 |
| His–Ser < 3.5 Å | 5.2% | **14.7%** | 0.0% | MUT-S8 |
| Attack distance | 3.67 Å | **3.29 Å** | 5.06 Å | **MUT-S8** |
| BD angle | 59.6° | **79.3°** | 46.7° | **MUT-S8** |
| Productive frames | 8.9% | **17.8%** | 0.2% | **MUT-S8** |
| Ca²⁺ coordination | Stable | Stable | Partial | WT-S8 / MUT-S8 |

**MUT-S8** shows the best overall catalytic geometry: shortest attack distance, BD angle closest to ideal, most productive frames, and the highest His–Ser contact frequency (14.7%, driven by replica_02 with 44%). This suggests the mutations enhance both nucleophile positioning and His–Ser triad organisation.

**WT-S8** has the best Asp–His contact (75.9%) and good attack geometry (8.9% productive), representing a well-organised Michaelis complex, though less productive than MUT-S8.

**ENG-S8** shows the worst performance: no His–Ser contact, poor attack geometry (5.06 Å, 0.2% productive), and only two structural Ca²⁺ ions. The engineered variant (Asp20–His52–Ser209) appears to have a significantly altered active site architecture that disrupts substrate positioning for catalysis.

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

**MUT-S8 Asp42 pKa = 8.03** is the notable exception. This elevation persists even without the substrate, indicating it is a direct consequence of the mutations increasing Asp42 burial (propKa predicts 100% desolvation for Asp42 in MUT-S8 vs a smaller contribution in WT-S8). If correct, Asp42 in MUT-S8 would be **protonated at pH 7.0**, which would impair its role as the general base in the charge relay. This is consistent with the weak His–Ser triad contact seen in MUT-S8 (only 0.6% of frames below 3.5 Å).

### Per-nitrogen distance analysis

To assess whether the simulated HID tautomer is geometrically consistent with the observed dynamics, all four His nitrogen–heavy atom distances were tracked across all replicas and frames:

VMD inspection of the original input PDB confirmed **ND1 faces Asp, NE2 faces Ser** (canonical HID orientation). MDTraj relabels the stripped PDB as HIE, swapping ND1/NE2 atom names; all distances below use the physically correct atom assignment.

| Distance | WT-S8 | MUT-S8 | ENG-S8 |
|----------|-------|--------|--------|
| **ND1 – Asp(OD) mean (Å)** | **3.34 ± 1.03** | **4.07 ± 2.06** | **3.87 ± 1.57** |
| **ND1 – Asp(OD) < 3.5 Å** | **75.9%** | **71.9%** | **62.9%** |
| NE2 – Ser(OG) mean (Å) | 5.25 ± 0.96 | 5.62 ± 1.57 | 5.69 ± 1.29 |
| NE2 – Ser(OG) < 3.5 Å | 5.2% | **14.7%** | 0.0% |

### Interpretation

VMD inspection of the original input PDB confirmed the **canonical HID orientation: ND1 faces Asp, NE2 faces Ser**. This is consistent with the standard serine protease mechanism where ND1-H hydrogen-bonds to Asp(OD) and NE2 (free) accepts H from Ser-OH to activate the nucleophile.

The distance analysis confirms this geometry holds during the MD:
- **ND1–Asp interaction is well-maintained** in 63–76% of frames — a stable H-bond between the protonated ND1 and the Asp carboxylate
- **NE2–Ser contact is weak** (mean 5.2–5.7 Å), consistent with a pre-catalytic Michaelis complex where Ser has not yet been activated

**MUT-S8** is the only system where NE2 reaches Ser (14.7% of frames < 3.5 Å, replica_02), correlating with its higher productive frame count (17.8%). **ENG-S8** shows no NE2–Ser contact (0%), consistent with its failure to achieve productive attack geometry.

**Note on MDTraj label swap:** MDTraj relabels HID as HIE when saving stripped PDB files, swapping the ND1 and NE2 atom names in the output. All distances in this report use the physically correct assignment verified by VMD (ND1→Asp, NE2→Ser). The corrected analysis used `his_acceptor="NE2"` and `his_donor="ND1"` in `compute_triad_geometry` to account for this artifact.

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
