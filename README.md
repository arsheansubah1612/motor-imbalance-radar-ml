
# Robust Multi-Condition Motor Imbalance Diagnosis via Non-Contact Coherent Radar Sensing

> **Status:** Manuscript under review at *IEEE Transactions on Industrial Informatics* (2026).
> Full methodology, feature design, and code will be released upon acceptance.

A non-contact machine learning framework for diagnosing rotor imbalance in rotating machinery, using 24 GHz continuous-wave radar I/Q signals and deployment-aware learning. The system reaches **98.7% accuracy** with a **~1 MB model footprint**, and holds **≥88% accuracy across all tested operating conditions**, making it suitable for edge deployment in industrial monitoring.

---

## Motivation

Rotor imbalance is one of the most common faults in rotating machinery. Conventional diagnosis relies on contact-mounted accelerometers or electrical current signature analysis — approaches that are hard to scale across industrial fleets, and that often fail when the machine's speed or load varies in real deployment.

This project asks a different question: can a non-contact radar system, combined with deployment-aware ML, provide equally reliable imbalance diagnosis across the full operating range of a real industrial drive? The answer, from this work, is yes — if the pipeline is designed around the physics of radar micro-vibration sensing, not just the machine-learning model.

---

## Approach

The pipeline treats each complete 30-second radar acquisition as a single diagnostic sample. This design choice preserves low-frequency modulation, harmonic structure, and long-range statistical characteristics that short-window segmentation would otherwise discard. Classification is performed at the record level, which also prevents data leakage between training and testing partitions.

Raw I/Q signals are first preprocessed into magnitude, phase, and instantaneous-frequency representations. From these, a 110-dimensional handcrafted feature vector is extracted, spanning time, spectral, cepstral, wavelet, and entropy domains. A domain-informed selection stage then compresses this into a compact 64-feature subset by removing redundant and physically uninformative descriptors. The reduced feature vectors are passed to Bayesian-optimized classical ML models — a Gaussian SVM and an ensemble learner — tuned over 200 Bayesian optimization iterations with 5-fold cross-validation.

The result is a four-class diagnosis (Normal, 10 g, 20 g, 30 g imbalance) that generalizes across both the 11 tested rotational speeds and the 6 tested load conditions.

---

## Dataset

The work uses the publicly available **Radar-Based Rotational Imbalance Detection Dataset** published on IEEE DataPort.

| Parameter | Value |
|---|---|
| Radar type | 24 GHz continuous-wave (CW) |
| Sampling frequency | 10 kHz |
| Recording duration | 30 s per record |
| Samples per channel | 300,000 |
| Signal channels | In-phase (I) and Quadrature (Q) |
| Sensor distance | 30 cm |
| Rotational speed range | 500 – 1500 rpm (11 levels) |
| Load range | 0 – 3 Nm (6 levels) |
| Total recordings | 1,802 |
| Imbalance classes | 4 (Normal / 10 g / 20 g / 30 g) |

The dataset exercises the diagnosis pipeline across **66 operating condition combinations** (11 speeds × 6 loads), making it well-suited for studying real-deployment robustness rather than single-condition performance.

---

## Headline results

### Overall classification performance

Evaluated on the combined all-RPM/all-load dataset under 5-fold cross-validation:

| Model | Features | Accuracy | Macro-F1 | Inference Speed | Model Size |
|---|---|---|---|---|---|
| **Optimized Gaussian SVM** | **64** | **98.7%** | **98.6%** | **~44,000 obs/s** | **~1 MB** |
| Optimized Ensemble | 64 | 98.2% | 98.2% | ~2,100 obs/s | ~17 MB |

The Gaussian SVM did not only achieve the highest accuracy, but did so with roughly **17× smaller footprint** and **20× faster inference** than the ensemble — a combination that matters for real industrial deployment where models often need to run on edge hardware with constrained memory and compute budgets.

### Robustness across operating conditions

The framework was explicitly stress-tested beyond a single aggregate accuracy number:

| Evaluation protocol | Result |
|---|---|
| Constant-RPM analysis (per-speed, mixed load) | ≥88% accuracy at every tested RPM, peak 96.5% at 1200 rpm |
| Constant-load analysis (per-load, mixed speed) | 85.5% – 90.4% across all load levels |
| **Leave-one-RPM-out cross-generalization** | **94.3% mean accuracy** (64-feature subset) |

The compact 64-feature subset retained nearly identical generalization performance to the full 110-feature set (94.3% vs. 94.6% mean leave-one-RPM-out accuracy — a difference of only 0.3 points), suggesting that handcrafted feature selection also acts as a form of regularization.

### Comparison with prior work

Compared on the same dataset against the CMES baseline (Alamri et al., 2026, *Computer Modeling in Engineering & Sciences*):

| Pipeline | Initial Features | Selected Features | Reported Performance |
|---|---|---|---|
| CMES — Extra Trees | 65 | 20 | 98.00% CV accuracy |
| CMES — SVM-RBF | 65 | 20 | 95.56% CV accuracy |
| **This work — SVM** | **110** | **64** | **98.7% validation accuracy** |

The proposed pipeline matches or exceeds the best prior result on this dataset while using a lighter classifier and a broader feature representation — evidence that moderately-reduced feature spaces can offer a better balance between compactness and information preservation than aggressive dimensionality reduction.

---


## Feature philosophy

Rather than relying on a single signal view such as a spectrogram, FFT, or time-domain window alone, the feature design integrates complementary physical descriptions of the radar return. I/Q channel statistics capture baseline amplitude and inter-channel consistency. Magnitude-domain features summarize envelope behavior. Phase-dynamics descriptors characterize unwrapped phase and its temporal derivatives. Instantaneous-frequency statistics capture modulation strength and non-stationary dynamics. Spectral-shape descriptors (centroid, roll-off, flatness, entropy) and band-power features describe how energy is distributed across diagnostic frequency bands. Harmonic descriptors — critical for imbalance signatures — track fundamental and harmonic ratios, since imbalance severity systematically alters this structure. Cepstral features capture periodicity in the log-spectral domain. Wavelet energy and permutation entropy complete the picture by characterizing multi-resolution energy distribution and ordinal complexity.

The design principle is simple: imbalance leaves fingerprints in multiple signal views at once, and a classifier that sees all of them simultaneously generalizes better than one that sees only one.


## Why this matters for industrial deployment

Three properties set this pipeline apart from a typical ML-on-signals paper. First, no contact sensors are required — no mounting, no cabling, no mechanical coupling; the radar sits 30 cm from the machine. Second, the model is robust to operating condition shifts, validated under constant-RPM, constant-load, and leave-one-RPM-out protocols that reflect the kinds of distribution shifts which break naive ML pipelines in real plants. Third, it is edge-deployable: a 1 MB model running ~44,000 inferences per second on constrained hardware, without needing a server.

Together, these properties move the work beyond a laboratory demonstration and toward something usable in operational industrial monitoring.



## Repository structure

```
├── README.md                    (this file)
├── figures/                     (selected result figures)
│   ├── pipeline_overview.png
│   ├── confusion_matrix.png
│   ├── robustness_analysis.png
│   └── per_load_confusion.png
└── LICENSE
```

Full code and detailed hyperparameter configurations will be released after acceptance of the manuscript, in accordance with IEEE's pre-publication policy.

---

Dataset credit:


Y. E. Acar. "Radar-based rotational imbalance detection dataset."
IEEE DataPort, 2025.





