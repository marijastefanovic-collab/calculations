# MD Simulation Analysis Report
## ENG-S8 — Catalytic Geometry with LPYPQPQLP

**Date:** 2026-06-04  
**Analysis by:** Marija Stefanovic  

---

## 1. Overview

This report summarises the catalytic geometry analysis of MD simulations of **ENG-S8** in complex with the gluten-derived peptide **LPYPQPQLP**. The system was prepared from the best AlphaFold3 model (seed-4_sample-0, ipTM = 0.69) and simulated using the same protocol as the BSC glutenase pipeline (ff14SB/TIP3P, 310 K, 3 × 500 ns).

### System

| Property | Value |
|----------|-------|
| Enzyme | ENG-S8 |
| Sequence length | 263 aa |
| Substrate | LPYPQPQLP (P1 = Gln5) |
| Structural Ca²⁺ | 1 × Ca²⁺ (Ca272) |
| Structure source | AlphaFold3 (seed-4_sample-0) |
| Ranking score | 0.75 |

### Active Site

| Residue | Role |
|---------|------|
| Asp20 | Catalytic Asp (triad base) |
| His52 | Catalytic His (HID: ND1→Asp, NE2→Ser) |
| Ser209 | Catalytic Ser (nucleophile) |
| Gln267 | P1 substrate residue (Gln5 of LPYPQPQLP) |
| Ca272 | Structural Ca²⁺ |

### Simulation Parameters

| Parameter | Value |
|-----------|-------|
| Force field | ff14SB |
| Water model | TIP3P |
| Temperature | 310 K |
| pH (protonation) | 8.5 (propKa 3.5.1) |
| His52 state | HID (pKa 7.62 < 8.5 → neutral) |
| Integrator | Langevin Middle, 2 fs time step |
| Pressure | 1 bar (Monte Carlo barostat) |
| Production | 5 × 100 ns per replica |
| Replicas | 3 |
| Total simulation time | 1,500 ns |
| Total frames analysed | 15,000 |

---

## 2. Catalytic Triad Geometry

The triad hydrogen bond network was monitored via two distances:
- **Asp20(OD) – His52(ND1):** proton acceptor arm
- **His52(NE2) – Ser209(OG):** proton donor arm (Ser activation)

A functional H-bond is < 3.5 Å.

### Results per Replica

| Replica | Asp–His mean (Å) | His–Ser mean (Å) | His–Ser < 3.5 Å |
|---------|------------------|------------------|------------------|
| rep1 | 5.37 ± 0.90 | 5.59 ± 1.26 | 4.7% |
| rep2 | 5.92 ± 1.17 | 5.95 ± 1.34 | 1.0% |
| rep3 | 5.01 ± 0.28 | 5.64 ± 1.23 | 0.2% |
| **All replicas** | **5.43 ± 0.97** | **5.73 ± 1.28** | **2.0%** |

### Interpretation

Both triad distances remain well above the 3.5 Å H-bond threshold in all replicas, consistent with an unorganised pre-catalytic state. The Asp–His arm never forms a persistent contact (0.0% < 3.5 Å). His–Ser contact is rare (2.0% overall), with replica 1 showing the most frequent sampling (4.7%). The high Asp–His mean in replica 2 (5.92 Å) indicates partial triad disengagement.

His–Ser contact is rare (2.0% overall) and Asp–His contacts are absent, indicating the triad geometry is not optimally configured for catalysis in this simulation.

---

## 3. Tetrahedral Attack Geometry

The nucleophilic attack of Ser209(OG) on the carbonyl carbon of the P1 residue (Gln5) was monitored via:
- **Attack distance:** Ser209(OG) – Gln5(C), productive threshold < 4.0 Å
- **Bürgi–Dunitz (BD) angle:** Nu–C=O, productive range 90°–120°

A frame is **productive** when both criteria are met simultaneously.

### Results per Replica

| Replica | Attack distance (Å) | BD angle (°) | Productive frames |
|---------|---------------------|--------------|-------------------|
| rep1 | 5.60 ± 0.62 | 69.0 ± 14.3 | 0.0% |
| rep2 | 5.34 ± 0.72 | 67.5 ± 19.8 | 0.0% |
| rep3 | 4.88 ± 0.64 | 54.6 ± 20.3 | 0.0% |
| **All replicas** | **5.27 ± 0.73** | **63.7 ± 18.6** | **0.0%** |

### Interpretation

No productive attack geometry was observed in any replica. The mean attack distance (~5.3 Å) and BD angle (~64°, far below the productive 90°–120° range) are both unfavourable. Replica 3 shows the shortest attack distance (4.88 Å), suggesting some tendency toward closer approach, but the angle remains inadequate.

The complete absence of productive frames indicates that the LPYPQPQLP peptide does not achieve a catalytically competent pose in the ENG-S8 active site over 1,500 ns of combined simulation.

---

## 4. Oxyanion Formation Potential

The oxyanion hole interaction was monitored as the distance between the backbone NH of Ser209 and the carbonyl oxygen of the P1 Gln5 residue. Formation of this H-bond (< 3.5 Å) stabilises the tetrahedral intermediate during catalysis.

### Results per Replica

| Replica | Ser209(N)→Gln5(O) mean (Å) | < 3.5 Å |
|---------|----------------------------|---------|
| rep1 | 7.45 ± 0.75 | 0.0% |
| rep2 | 7.04 ± 1.10 | 0.0% |
| rep3 | 6.30 ± 1.02 | 0.0% |
| **All replicas** | **6.93 ± 1.02** | **0.0%** |

### Interpretation

The oxyanion hole is not engaged in any replica. Distances of 6–7.5 Å indicate the substrate carbonyl is far from the Ser209 backbone NH, consistent with the poor attack geometry observed above. This rules out even transient oxyanion hole formation in the current simulations.

---

## 5. Ca²⁺ Ion Coordination

Subtilisin E contains a single structural Ca²⁺ (Ca272). First-shell coordination was calculated using a 2.9 Å cutoff, counting protein O/N donor atoms.

### Results

| Ion | rep1 | rep2 | rep3 | Mean |
|-----|------|------|------|------|
| Ca272 | 6.9 ± 0.3 | 6.7 ± 0.5 | 6.7 ± 0.5 | **6.8 ± 0.4** |

### Interpretation

Ca272 is stably coordinated throughout all three replicas (mean 6.8 ± 0.4), consistent with a well-defined structural Ca²⁺ site with octahedral or higher coordination. Ca²⁺ does not dissociate or lose coordination during the simulation, indicating the structural scaffold is stable.

---

## 6. Summary

| Metric | ENG-S8 |
|--------|--------|
| Asp–His < 3.5 Å | 0.0% |
| His–Ser < 3.5 Å | 2.0% |
| Attack distance mean (Å) | 5.27 ± 0.73 |
| BD angle mean (°) | 63.7 ± 18.6 |
| Productive frames | 0.0% |
| Oxyanion < 3.5 Å | 0.0% |
| Ca272 coordination | 6.8 ± 0.4 (stable) |

ENG-S8 does not achieve productive attack geometry toward LPYPQPQLP in any of the three replicas over 1,500 ns of combined simulation. The triad is disengaged (no Asp–His contacts, rare His–Ser), the nucleophile is too far from the P1 carbonyl (~5.3 Å), the BD angle is unfavourable (~64°), and the oxyanion hole is not engaged. The structural Ca²⁺ site is stable throughout. These results establish the pre-engineering baseline and confirm that active site engineering will be required to improve substrate positioning.

---

## 7. Figures

| Figure | Description |
|--------|-------------|
| fig1_triad_geometry.png | Violin plots — Asp–His and His–Ser distance distributions per replica |
| fig2_triad_timeseries.png | Triad distances over simulation time |
| fig3_attack_geometry.png | Violin plots — attack distance and BD angle per replica |
| fig4_attack_landscape.png | 2D hexbin — attack distance vs BD angle with productive zone |
| fig5_ca_coordination.png | Ca272 coordination count over time |
| fig6_oxyanion.png | Oxyanion distance distribution and timeseries |

All figures: `subtilsin_analysis_results/plots/`

---

## 8. Analysis Pipeline

```
Structure:  AlphaFold3 best model (seed-4_sample-0)
            mature_subtilsin_best.pdb
Protonation: propKa 3.5.1, pH 8.5 → His52 = HID (pKa 7.62)
MD setup:   tleap (ff14SB, TIP3P, 12 Å box, neutralisation)
            mature_subtilsin.prmtop / mature_subtilsin.inpcrd
Production: OpenMM, 3 × 5 × 100 ns on MN5
Stripping:  strip_subtilsin.py (keeps protein + peptide + Ca2+)
            Output: replica_0X/production_0X_stripped.dcd
Topology:   md/minimized_non_solvent.pdb
Analysis:   subtilsin_analysis.py (md_enzyme_analysis)
            — compute_triad_geometry (his_acceptor="ND1", his_donor="NE2")
            — compute_tetrahedral_attack (productive: d<4.0Å AND BD 90°–120°)
            — compute_ion_coordination (Ca272, cutoff 2.9 Å)
            — compute_donor_acceptor_distance (oxyanion: Ser209-N → Gln5-O)
Output:     subtilsin_analysis_results/
            triad_geometry.xlsx, tetrahedral_attack.xlsx,
            ca_coordination.xlsx, oxyanion.xlsx, plots/
```
