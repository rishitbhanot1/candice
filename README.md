<div align="center">

# CANDICE
## CAN-based Anomaly and Network Detection for Intelligent Cybersecurity Evaluation
**A research repository for CAN bus intrusion detection, model experimentation, and edge-oriented deployment.**

[![Domain](https://img.shields.io/badge/domain-automotive%20cybersecurity-FFFFFF)](#about-the-lab)
[![Focus](https://img.shields.io/badge/focus-CAN%20intrusion%20detection-000000)](#research-areas)
[![Implementations](https://img.shields.io/badge/implementations-versioned-5B6B7A)](#implementation-index)

</div>

---

## Overview

The **CAN Automotive Intrusion Detection Lab** is a centralized collection of independent implementations exploring how malicious or anomalous activity in automotive Controller Area Network (CAN) traffic can be detected.

The repository is organized around **versioned experiments**. Each version can use a different detection method, dataset, model architecture, feature representation, optimization technique, or deployment target. Implementations may be delivered as Jupyter notebooks, Python packages/scripts, model artifacts, firmware, or a combination of these.

The aim is to make experiments easier to explore, reproduce, compare, and extend—without forcing every approach into the same code structure or software environment.

> **Repository status:** Research collection under active development.
> 
> The implementation index distinguishes available work from planned directions.

## At a glance

| | |
|---|---|
| **Domain** | Automotive cybersecurity |
| **Network focus** | Controller Area Network (CAN) |
| **Primary objective** | Intrusion and anomaly detection |
| **Implementation formats** | Jupyter notebooks, Python, model artifacts, embedded code |
| **Research scope** | Classical ML, deep learning, TinyML, optimization, deployment |
| **Organization** | Independent, versioned implementations |

## Research areas

The repository can accommodate work across the following areas. These are the lab's areas of interest, not a claim that every area is already implemented.

| Research area | Example topics |
|---|---|
| Classical machine learning | Feature-based classification, Random Forest, SVM, gradient boosting |
| Deep learning | Dense networks, CNNs, RNNs, LSTMs, autoencoders |
| Anomaly detection | Supervised, semi-supervised, and unsupervised detection |
| TinyML and edge AI | Compact neural networks and resource-constrained inference |
| Model optimization | Quantization-aware training, post-training quantization, pruning, compression |
| Embedded deployment | Model conversion, hardware integration, inference on edge platforms |
| Real-time monitoring | Online CAN frame processing and detection pipelines |
| Evaluation | Detection performance, false alarms, latency, memory, model size, robustness |

## Implementation index

Implementations are maintained under [`versions/`](versions/). Each version's own `README.md` is the source of truth for its methodology, setup, execution instructions, dependencies, artifacts, results, and limitations.

| Version | Implementation | Main focus | Status |
|---|---|---|---|
| [V01 — TinyML QAT INT8](versions/v01-tinyml-qat-int8/) | TinyML CAN IDS with quantization-aware training | Neural-network classification and INT8 TensorFlow Lite conversion | Available |

**Status definitions**

| Status | Meaning |
|---|---|
| Available | Implementation and its documentation are present in the repository |
| Initial implementation | First version has been added; results and documentation may continue to evolve |
| In Progress| Script has been updated to the latest set; results and documentation till the last update have been published |

### What belongs where?

| Path | Responsibility |
|---|---|
| `versions/` | Self-contained CAN IDS implementations and experiments |
| `versions/<version>/notebooks/` | Exploratory work, experiments, and notebook-based workflows |
| `versions/<version>/src/` | Reusable Python modules and executable scripts |
| `versions/<version>/configs/` | Experiment settings and preprocessing metadata |
| `versions/<version>/models/` | Selected trained models and deployment artifacts |
| `versions/<version>/results/` | Version-specific metrics, figures, and comparisons |
| `shared/` | Stable utilities genuinely reused across multiple versions |
| `datasets/` | Dataset sources, access instructions, and licensing notes—not raw datasets by default |
| `docs/` | Repository-wide methodology and contribution conventions |
| `results/` | Optional cross-version summaries and comparisons |
| `assets/` | Repository-level diagrams and figures |
| `.github/` | Issue templates and repository automation |

## Getting started

There is no single environment for the entire repository. Dependencies and execution steps are defined separately for each implementation.

1. Browse the [implementation index](#implementation-index).
2. Open the selected version's `README.md`.
3. Follow its dataset and environment setup instructions.
4. Run the documented notebook or scripts.
5. Review the evaluation protocol, results, and limitations.

For the first implementation, start here:

```text
versions/v01-tinyml-qat-int8/
```

Do not assume that commands or dependencies from one version apply to another.

## Datasets and data handling

Each implementation should document its dataset name, original source, version or release where available, access procedure, relevant license or terms, and any preprocessing applied.

**Raw datasets should not be committed to the repository by default.** Prefer downloading from the original provider or using an approved data-hosting mechanism. Do not redistribute datasets or derived artifacts unless the applicable terms allow it.

See [`datasets/README.md`](datasets/README.md) for repository-wide dataset guidance.

## Reproducibility and evaluation

Each implementation should provide enough context for another researcher to understand the experiment and reproduce it where practical.

| Document | What to record |
|---|---|
| Objective | Detection task, threat or anomaly scope, intended use |
| Dataset | Name, source, version, access conditions, license |
| Data preparation | Filtering, labeling, feature extraction, normalization, train/test split |
| Method | Detection algorithm, model architecture, key hyperparameters |
| Environment | Python and library versions; hardware where relevant |
| Execution | Commands, notebook order, configuration files |
| Evaluation | Metrics, test protocol, class-wise results, limitations |
| Artifacts | Model format, preprocessing metadata, output files |

For classification tasks, include class-wise precision, recall, and F1-score where appropriate—not only overall accuracy. For edge or deployment-oriented experiments, report measured model size, memory use, inference latency, or throughput when available.

Keep the following distinctions clear:

- **Measured:** directly obtained from a documented experiment.
- **Estimated:** calculated or inferred, but not directly measured.
- **Planned:** intended future evaluation or work.

Comparisons between versions are meaningful only when dataset selection, preprocessing, data splits, and evaluation conditions are sufficiently aligned. State differences explicitly.

## Disclaimer

This repository is intended for educational & research purposes. Implementations and experimental results are research artifacts; they are not, by themselves, certified automotive security products.

Testing on a vehicle or production network should only be performed with appropriate authorization, controlled equipment, and suitable safety precautions. Offline datasets and simulated environments are preferable for initial experimentation.

<!-- ## License

// The repository license is specified in [`LICENSE`](LICENSE). Datasets, third-party code, pretrained models, and other external artifacts may have separate terms. Users are responsible for reviewing and complying with those terms.
-->
---

<div align="center">

**CAN Automotive Intrusion Detection Lab**

*Explore · Experiment · Evaluate · Extend*

</div>
