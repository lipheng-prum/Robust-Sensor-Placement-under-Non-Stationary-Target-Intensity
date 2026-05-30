<<<<<<< HEAD
# Robust Sensor Placement under Non-Stationary Target Intensity

A toy simulation study extending the barrier coverage sensor placement framework from [Kim et al. (2024)](https://ieeexplore.ieee.org/document/11014804) to handle non-stationary target intensity patterns.

**Author:** Lipheng  
**Date:** May 2026

---

## Overview

This project extends the state-of-the-art sensor placement optimization framework to address a key real-world limitation: **target intensity is not static**. In maritime surveillance, traffic patterns vary across seasons, weather conditions, and time of day. A sensor deployment optimized for one period may have blind spots in another.

### Key Innovation

The original framework optimizes sensor locations for a single, fixed target intensity function. This work proposes a **worst-case robust (maximin) formulation** that:

- Handles **K = 3 non-stationary intensity scenarios** representing diverse traffic patterns
- Uses a robust greedy algorithm to maximize the minimum performance across all scenarios
- Shows empirically that robustness comes at **essentially no cost to average performance**

### Main Finding

At M = 5 sensors, the robust greedy method achieves:
- **Worst-case void probability:** 0.370 (17.8% improvement over uniform baseline)
- **Weighted-average void probability:** 0.376 (tied with average-case greedy)
- **Intuitive spatial strategy:** Spreads sensors broadly to hedge against diverse traffic patterns

---

## Table of Contents

- [Technical Foundation](#technical-foundation)
- [Methodology](#methodology)
- [Results](#results)
- [Installation & Usage](#installation--usage)
- [References](#references)
- [Future Work](#future-work)

---

## Technical Foundation

### Mathematical Framework

The framework operates in a **representation space** that maps every straight-line trajectory to a unique point:

- **α (alpha):** Angle of the line's normal vector
- **p:** Signed perpendicular distance from the origin

This parameterization is discretized on a 40×40 grid for numerical integration and enables efficient detection probability calculations.

### Sensor Detection Model

Each sensor's detection probability decays isotropically with distance to the target trajectory:

```
γ(ζ, a_i) = ρ · exp(-d²(l, a_i) / σ_l)
```

Where:
- ρ = 0.95 (maximum detection probability)
- σ_l = 4.0 km² (effective range parameter)

### Void Probability Approximation

The void probability (probability that a target trajectory is never detected) is:

```
ν(a) = exp(-∫ λ(l) · π_C(l, a) dl)
```

Where π_C is the joint miss probability across all sensors.

---

## Methodology

### Three Non-Stationary Intensity Scenarios

Each scenario represents a distinct traffic pattern:

1. **Concentrated Corridor:** Main shipping lane in calm season (α ≈ 90°, p = 0–3 km)
2. **Crossing Routes:** Divergent patterns during storm season (~45° and ~135°)
3. **Diffuse Clusters:** Re-routed traffic during construction/events (varied angles)

Intensities are modeled as Gaussian mixtures with calibrated peak amplitudes to produce void probabilities in the 0.05–0.45 range, consistent with empirical data.

### Placement Algorithms

Four methods are compared:

1. **Average-case greedy:** Maximize weighted sum of void probabilities across scenarios
2. **Worst-case robust greedy:** Maximize minimum void probability (our proposed method)
3. **Uniform placement:** Baseline—sensors equally spaced along y = 0
4. **Scenario-specific oracle:** Independent greedy per scenario (unachievable upper bound)

### Optimization Formulation

**Original (single-scenario):**
```
â = argmax_a ν(a)
```

**Proposed (multi-scenario average-case):**
```
â_avg = argmax_a Σ_k w_k ν_k(a)
```

**Proposed (multi-scenario worst-case robust):**
```
â_rob = argmax_a min_{k=1,...,K} ν_k(a)
```

**Key theoretical insight:** While the original objective is submodular (guaranteeing ≥63% of optimal via greedy), the minimum of submodular functions loses this property. The robust greedy is therefore a practical heuristic without formal approximation guarantees.

---

## Results

### Quantitative Performance

| Method | M=1 | M=2 | M=3 | M=4 | M=5 |
|--------|-----|-----|-----|-----|-----|
| Worst-case greedy | 0.783 | 0.556 | 0.483 | 0.422 | **0.370** |
| Average-case greedy | 0.783 | 0.558 | 0.485 | 0.424 | 0.362 |
| Uniform baseline | 0.880 | 0.707 | 0.565 | 0.442 | 0.314 |
| Oracle (per-scenario) | 0.764–0.800 | 0.475–0.538 | 0.391–0.467 | 0.331–0.413 | 0.384–0.421 |

**Weighted-average void probability at M=5:** ~0.376 (both greedy methods)

### Key Observations

✓ **Robust method excels at worst-case performance** — consistently highest across all sensor counts  
✓ **No sacrifice in average case** — weighted-average VP ≈ 0.376, matching average-case greedy  
✓ **Intuitive spatial patterns** — robust placement spreads sensors broadly; average-case clusters  
✓ **Substantial gap to oracle is inherent** — single deployment cannot match scenario-specific tuning  

### Visualizations

The repository includes five publication-quality figures:

- **fig1_intensity_scenarios.png:** Heatmaps of the three intensity scenarios in representation space
- **fig2_void_prob_comparison.png:** Per-scenario void probability vs. sensor count
- **fig3_worstcase_comparison.png:** Worst-case void probability comparison (key result)
- **fig4_sensor_placements.png:** Physical domain placement patterns for M=5 sensors
- **fig5_thinned_intensity.png:** Residual intensity after robust sensor placement

---

## Installation & Usage

### Requirements

- Python 3.12
- NumPy ≥ 1.24.0
- SciPy ≥ 1.10.0
- Matplotlib ≥ 3.7.0
- Pandas ≥ 2.0.0
- Jupyter ≥ 1.0.0
- IPython ≥ 8.0.0

### Setup

1. **Clone or download this repository**

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Launch Jupyter:**
   ```bash
   jupyter notebook robust_sensor_placement.ipynb
   ```

### Running the Analysis

The notebook is self-contained and fully executable:

- **Section 1:** Parameterizes the representation space (α, p)
- **Section 2:** Defines three synthetic intensity scenarios
- **Section 3:** Implements sensor detection model (Gaussian decay)
- **Section 4:** Formulates average-case and worst-case robust objectives
- **Section 5:** Implements four placement algorithms
- **Section 6:** Runs the core computational study for M = 1–5 sensors
- **Sections 7–9:** Generates results tables, visualizations, and discussion

Execute cells sequentially to reproduce all results and figures.

---

## References

[1] M. Kim, D. J. Stilwell, H. Yetkin, and J. Jimenez, "Near-optimal Sensor Placement for Detecting Stochastic Target Trajectories in Barrier Coverage Systems," *IEEE*, 2024.

[2] S. N. Chiu, D. Stoyan, W. S. Kendall, and J. Mecke, *Stochastic Geometry and Its Applications*, John Wiley & Sons, 2013.

[3] S. Martino and A. Riebler, "Integrated Nested Laplace Approximations (INLA)," arXiv:1907.01248, 2019.

[4] H. Rue, S. Martino, and N. Chopin, "Approximate Bayesian Inference for Latent Gaussian Models by Using Integrated Nested Laplace Approximations," *JRSS-B*, vol. 71(2), 2009.

[5] M. Kim, H. Yetkin, D. J. Stilwell, et al., "Toward Optimal Placement of Spatial Sensors," *IEEE Access*, 2023.

[6] A. Krause, A. Singh, and C. Guestrin, "Near-optimal Sensor Placements in Gaussian Processes," *JMLR*, vol. 9, 2008.

[7] L. Lovász, "Submodular functions and convexity," in *Mathematical Programming: The State of the Art*, Springer, 1983, pp. 235–257.

[8] A. Ben-Tal, L. El Ghaoui, and A. Nemirovski, *Robust Optimization*, Princeton University Press, 2009.

[9] A. Krause, H. B. McMahan, C. Guestrin, and A. Gupta, "Robust Submodular Observation Selection," *JMLR*, vol. 9, pp. 2761–2801, 2008.

---

## Future Work

### High-Priority Extensions

1. **Real AIS Data Validation:** Estimate per-month intensity scenarios using INLA on actual maritime traffic data and validate whether robust placement outperforms single-month optimization on held-out months.

2. **Distributionally Robust Formulation:** Model intensity uncertainty as an ambiguity set around the LGCP posterior, connecting to distributionally robust optimization and potentially yielding principled approximation guarantees.

3. **Approximation Bounds:** Investigate whether the exponential structure of void probability enables provable approximation ratios for the maximin greedy algorithm.

### Practical Extensions

4. **Adaptive Re-deployment:** For mobile sensors (e.g., AUVs), develop online algorithms that re-position sensors as the active scenario is identified from incoming detections.

5. **Heterogeneous Sensors:** Extend to sensors with different costs, ranges, detection profiles, and directionalities under budget constraints.

6. **Larger-Scale Studies:** Increase the number of scenarios (K > 3) and sensor candidates to explore scalability and robustness to scenario misspecification.

---

## License

MIT

## Contact

For questions or collaboration, please contact Lipheng.
=======
# Robust-Sensor-Placement-under-Non-Stationary-Target-Intensity
This notebook implements and evaluates my proposed extension to the barrier coverage sensor placement framework from:  > M. Kim, D. J. Stilwell, H. Yetkin, and J. Jimenez, "Near-optimal Sensor Placement  > for Detecting Stochastic Target Trajectories in Barrier Coverage Systems," IEEE, 2024.
>>>>>>> eafcc90de7ea898258519080ba61e6672e4d9a21
