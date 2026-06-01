# MD Simulation Analysis Report
## Glutenase Variants — Catalytic Geometry Analysis

**Date:** 2026-06-01  
**Data source:** Martin Floor, BSC — `/gpfs/projects/bsc72/mfloor/glutenase/phase_a/ff14SB`  
**Analysis by:** Marija Stefanovic  

---

## 1. Overview

This report summarizes the catalytic geometry analysis of glutenase MD simulations from the BSC glutenase project. Two enzyme variants with a bound substrate peptide (**LPYPQPQLP**) were analysed over ~500 ns of production MD per replica.

### Systems Analysed

| System | Description | Replicas | Simulation time | Frames |
|--------|-------------|----------|-----------------|--------|
| WT-S8 | Wild-type glutenase + LPYPQPQLP | 3 | ~480 ns each | 14,371 |
| MUT-S8 | Mutant glutenase + LPYPQPQLP | 3 | ~490 ns each | 14,679 |
| REF-S9 | Reference enzyme (different family) | 3 | ~100 ns each | — |
| ENG-S8 | Engineered variant (no stripped traj.) | 3 | — | — |

> **Note:** REF-S9 belongs to a different protease family with a distinct active site numbering — catalytic triad and attack geometry analyses were not performed for this system. ENG-S8 trajectories had not been water-stripped at the time of analysis.

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

| Residue | Role | Biological number |
|---------|------|------------------|
| Asp42 | Catalytic Asp | 42 |
| His116 | Catalytic His | 116 |
| Ser288 | Catalytic Ser (nucleophile) | 288 |
| Gln355 | P1 substrate residue | 355 (chain A) |
| Ca360–362 | Structural Ca²⁺ ions | 360–362 |

---

## 2. Catalytic Triad Geometry

The catalytic triad hydrogen bond network was monitored via two distances:
- **Asp42(OD) – His116(ND1):** proton acceptor arm of the triad
- **His116(NE2) – Ser288(OG):** proton donor arm of the triad (activates the nucleophilic Ser)

A functional H-bond is typically < 3.5 Å.

### Summary Statistics (all replicas combined)

| Metric | WT-S8 | MUT-S8 |
|--------|-------|--------|
| Asp–His mean distance (Å) | 4.98 ± 0.49 | 5.03 ± 0.71 |
| Asp–His median (Å) | 4.92 | 4.86 |
| His–Ser mean distance (Å) | 5.10 ± 0.98 | 5.75 ± 0.90 |
| His–Ser median (Å) | 5.16 | 5.68 |
| Frames with Asp–His < 3.5 Å | 0.0% | 0.0% |
| Frames with His–Ser < 3.5 Å | **8.8%** | 0.6% |

### Interpretation

Both systems show triad distances considerably above the H-bond threshold (~2.7–3.5 Å). Neither the Asp–His nor the His–Ser contact forms a persistent H-bond in the simulations. This is consistent with a **Michaelis complex** or **pre-catalytic state** where the triad geometry is not yet fully organised for catalysis.

WT-S8 shows slightly more frequent His–Ser contacts (8.8% of frames < 3.5 Å) compared to MUT-S8 (0.6%), suggesting the wild-type His–Ser interaction is marginally better maintained.

---

## 3. Tetrahedral Attack Geometry

The nucleophilic attack of Ser288 OG on the carbonyl carbon of the P1 residue (Gln355) was monitored via:
- **Attack distance:** Ser288(OG) – Gln355(C), productive threshold < 4.0 Å
- **Bürgi–Dunitz (BD) angle:** Nu–C=O angle, productive range 90°–120°

A frame is classified as **productive** when both criteria are met simultaneously (distance < 4.0 Å AND BD 90°–120°).

### Summary Statistics

| Metric | WT-S8 | MUT-S8 |
|--------|-------|--------|
| Attack distance mean (Å) | 3.67 ± 0.59 | **3.29 ± 0.38** |
| BD angle mean (°) | 59.6 ± 25.0 | **79.3 ± 11.2** |
| Productive frames (total) | 8.9% | **17.8%** |
| Productive — replica 1 | 14.5% | 10.6% |
| Productive — replica 2 | 0.2% | 0.6% |
| Productive — replica 3 | 12.0% | **42.1%** |

### Interpretation

**MUT-S8 shows significantly better attack geometry than WT-S8:**
- Mean attack distance is 0.4 Å shorter in MUT-S8 (3.29 vs 3.67 Å)
- BD angle is ~20° closer to the ideal 107° in MUT-S8 (79° vs 60°)
- MUT-S8 has twice the fraction of productive frames overall (17.8% vs 8.9%)
- MUT-S8 replica_03 achieves 42.1% productive frames, indicating an extended period with excellent attack geometry

The high inter-replica variability (particularly in WT-S8: 0.2–14.5%) suggests conformational sampling is incomplete in some replicas, and longer simulations or additional replicas may be needed to obtain converged estimates.

---

## 4. Ca²⁺ Ion Coordination

Three structural Ca²⁺ ions (resSeq 360–362) were monitored. The first-shell coordination count was calculated using a cutoff of 2.9 Å, counting protein O/N donor atoms (excluding bulk solvent and counter-ions).

### Mean Coordination Numbers

| Ca²⁺ ion | WT-S8 | MUT-S8 | Interpretation |
|----------|-------|--------|----------------|
| Ca360 | 6.8 ± 0.4 | 6.7 ± 0.5 | Well-coordinated, stable |
| Ca361 | 7.6 ± 0.5 | 7.4 ± 0.5 | Well-coordinated, stable |
| Ca362 | 3.2 ± 1.4 | 3.0 ± 1.1 | Partial / dynamic coordination |

### Interpretation

Ca360 and Ca361 are stably coordinated with ~7 ligands each, consistent with a fully occupied coordination shell typical of structural Ca²⁺ sites (coordination number 6–8). Ca362 shows notably lower and more variable coordination (mean ~3), suggesting it occupies a **solvent-exposed or partially occupied** site, and may exchange ligands with bulk water during the simulation.

No significant difference in Ca²⁺ coordination is observed between WT-S8 and MUT-S8, indicating the mutations do not perturb the structural calcium sites.

---

## 5. Summary and Conclusions

| Property | WT-S8 | MUT-S8 | Winner |
|----------|-------|--------|--------|
| Triad H-bond (His–Ser) | 8.8% | 0.6% | WT-S8 |
| Attack distance | 3.67 Å | 3.29 Å | **MUT-S8** |
| BD angle | 59.6° | 79.3° | **MUT-S8** |
| Productive frames | 8.9% | 17.8% | **MUT-S8** |
| Ca²⁺ coordination | Stable | Stable | — |

The mutant enzyme (MUT-S8) shows superior attack geometry compared to the wild-type across all three metrics (shorter attack distance, better BD angle, more productive frames). This suggests the mutations enhance the positioning of the nucleophilic Ser288 relative to the substrate carbonyl, which would be expected to increase catalytic efficiency.

However, the His–Ser triad contact is better maintained in WT-S8. This may reflect a trade-off: in MUT-S8, Ser288 is pulled closer to the substrate at the cost of a slightly weaker His–Ser interaction.

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

## 7. Analysis Pipeline

```
Input:  *_non_water.dcd + minimized_non_water.pdb
Tool:   md_catalytic_analysis.py (md_enzyme_analysis library)
Config: config_mfloor.json
Output: triad_geometry.xlsx, tetrahedral_attack.xlsx, ca_coordination.xlsx
```
