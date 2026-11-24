# ECG Waveform
A modular Python research framework for ECG PQRST fiducial point detection and delineation, signal quality assessment, and signal-quality-aware adaptive processing.

> **Please note that this is a research software.** This framework is intended for research and algorithm development only. It is not a medical device and must not be used for diagnosis or clinical decision-making.

## Overview

**ECG Waveform** is a Python-based research project on the automatic detection and delineation of fiducial points in electrocardiogram (ECG) signals: the onset, peak and offset of the P wave, the QRS complex and the T wave.

The project does not only compare detection algorithms. It investigates **why** detection errors occur, by relating the localization error of each fiducial point to the signal's quality, morphological and rhythm characteristics. Based on these findings, it develops and evaluates **adaptive processing strategies** that adjust preprocessing, parameters or the choice of detector to the input signal.

Three task levels are kept separate throughout:

1. **R-peak detection**
2. **P, Q, S and T wave localization**
3. **Full PQRST delineation** (onset, peak and offset points)

Not every algorithm provides every fiducial point, and not every database provides a reference for every point. Comparisons are therefore made only on points that all compared methods can output and for which a verified reference exists.

## Research Questions

| | Question |
|---|---|
| **RQ1** | How do the localization error (ms) and miss rate of each individual fiducial point vary as a function of noise type and signal-to-noise ratio (SNR)? Is there an interaction between these factors? |
| **RQ2** | Which morphological features are associated with individual detection error types after accounting for the effect of noise level? |
| **RQ3** | To what extent can detection errors exceeding the predefined tolerance threshold be predicted from signal quality indices (SQIs), morphological features, and rhythm features? How much does incorporating morphological and rhythm features improve predictive performance compared to a model relying exclusively on signal quality features? |
| **RQ4** | Does signal-adaptive preprocessing and parameter tuning improve localization accuracy compared to static processing, and does this improvement depend on signal quality? |
| **RQ5** | Does adaptive algorithm selection bring the detection error on individual signal segments closer to the best achievable error (i.e., the error of the best-performing detector for each segment) than any single detector applied uniformly across all segments? |
| **RQ6** | Does the advantage of the adaptive strategy persist when evaluated on a different dataset or under different recording conditions from those used to tune the strategy, restricting the analysis to points with verified reference annotations in both datasets? |

## Status

The project is developed incrementally. Its first focus is a modular ECG processing framework with reproducible evaluation. **Features listed in this README may not be available in the current version.**

## Features

- ECG signal loading and dataset-specific adapters
- Configurable signal preprocessing
- ECG signal quality assessment using SQI metrics
- Automatic detection of PQRST fiducial points
- Classical signal processing and wavelet-based delineation
- Machine learning-based detection approaches
- Synthetic ECG generation with configurable morphology, heart rate and noise
- Signal-quality-aware adaptive processing
- Confidence estimation and flagging of non-evaluable segments
- Quantitative evaluation against reference annotations, per fiducial point
- Reproducible, configuration-driven experiments and result visualization

```text
ecg-waveform/
├── configs/            # YAML experiment configurations
├── data/               # ECG databases (gitignored)
├── notebooks/          # Exploratory data analysis
├── scripts/            # Utility and database scripts
├── src/                # Main Python package
│   ├── core/           #   core entities, interfaces, and abstractions
│   ├── data/           #   data loaders and mappers
│   ├── signal/         #   quality, morphology, simulation
│   ├── detection/      #   detectors, adaptive, uncertainty
│   ├── evaluation/     #   metrics, robustness, statistics
│   ├── experiment/     #   experiment runner and logging
│   ├── utils/          #   shared utilities and helper functions
│   └── visualization/  #   visualization
├── tests/              # pytest tests
├── results/            # Generated tables and figures (gitignored)
└── docs/               # Project documentation (specification, designs)
```

The structure may evolve as the implementation develops.

## Installation

### Clone the repository

```bash
git clone https://github.com/gorozdi-attila/ecg-waveform.git
cd ecg-waveform
```

### Set up a virtual environment

**Linux / macOS**

```bash
python -m venv venv
source venv/bin/activate
```

**Windows**

```bash
python -m venv venv
venv\Scripts\activate
```

### Install the project

Core installation (classical methods, signal quality assessment, evaluation):

```bash
pip install -e .
```

Optional extras:

| Extra | Command | Contents |
|---|---|---|
| `ml` | `pip install -e ".[ml]"` | PyTorch, for the 1D CNN / 1D U-Net detectors |
| `viz` | `pip install -e ".[visualization]"` | Seaborn, additional plotting |
| `notebooks` | `pip install -e ".[notebooks]"` | Jupyter and ipykernel |
| `dev` | `pip install -e ".[dev]"` | pytest, pytest-cov, ruff, mypy |

Extras can be combined, for example `pip install -e ".[ml,dev]"`.

### Dependencies

See `pyproject.toml` for the complete and authoritative dependency list. The project is based on:

- Python 3.10+
- NumPy, SciPy, Pandas
- WFDB
- PyWavelets
- PyTorch
- scikit-learn
- PyYAML
- Matplotlib, Seaborn
- pytest (development)

## Datasets

The methods are evaluated on two publicly available ECG databases, plus synthetic signals for controlled experiments.

### MIT-BIH Arrhythmia Database

[**MIT-BIH Arrhythmia Database**](https://www.physionet.org/content/mitdb/1.0.0/)

The MIT-BIH Arrhythmia Database provides QRS (beat) annotations only. It is therefore used for QRS / R-peak detection evaluation. It has no reference annotations for P waves, T waves or wave boundaries.

### Lobachevsky University Electrocardiography Database (LUDB)

[**Lobachevsky University Electrocardiography Database**](https://physionet.org/content/ludb/1.0.1/)

LUDB contains 200 ECG records (12 leads, 10 s, 500 Hz) in which cardiologists manually annotated the boundaries and peaks of P waves, QRS complexes and T waves, separately for each lead. It supports the evaluation of PQRST delineation.

### Downloading the datasets

The databases are not included in this repo (due to its size). To download them:

```bash
python scripts/download_database.py mitdb
python scripts/download_database.py ludb
```

This downloads the records into the `data/mitdb/` and `data/ludb/` folders using the `wfdb` package.

## Detection Methods

The different approaches share a common detector interface so that their outputs and performance can be compared on equal terms. The interface distinguishes between a fiducial point a method **does not support** and one it **supports but did not detect**.

### Classical Signal Processing

- **Pan–Tompkins:** a QRS detection algorithm used for R-peak localization.
- **Wavelet-based delineation:** a wavelet-transform approach for locating the boundaries and peaks of the QRS complex, P wave and T wave (Martínez et al., 2004).

### Machine Learning

- **1D Convolutional Neural Networks (1D CNN)**
- **1D U-Net architectures**, for segmentation-based delineation

The methods are not treated as equivalent: they provide different fiducial points and are sensitive to different signal characteristics.

## Signal Quality Assessment and Adaptive Processing

An important research direction is how ECG signal characteristics influence detection accuracy. Three groups of characteristics are kept separate, because, for example, a low-amplitude but noise-free signal is not necessarily a poor-quality signal:

- **Signal quality:** noise level and type, baseline wander, powerline interference, high-frequency noise, motion and electrode artifacts, saturation and flatline segments.
- **Morphology:** wave amplitudes and ratios, widths, QRS duration, polarity, beat-to-beat variability.
- **Rhythm:** heart rate, RR variability, temporal proximity of waves.

Signal quality indices (SQIs) are intended to characterize signal quality and to support adaptive processing.

Potential adaptive approaches:

- selecting preprocessing based on signal characteristics (aggressive filtering can distort the P and T waves);
- adjusting detection parameters according to estimated signal quality;
- selecting the detection algorithm based on signal properties;
- comparing fixed and adaptive pipelines, including against an oracle that picks the best configuration per segment.

The objective is to determine whether adapting the processing strategy to the input signal improves detection accuracy and robustness. The benefit of adaptation is a hypothesis to be tested, not an assumption.

**Confidence.** Detected points can carry a reliability value. A heuristic score is distinguished from a calibrated confidence estimate, and the type of value is recorded with the result.

## Planned Evaluation

### Detection Performance

- **Sensitivity (Se) / Recall**
- **Positive Predictive Value (PPV) / Precision**
- **F1 score**
- **Detection Error Rate (DER)**, defined as (false + missed detections) / number of reference points

A detection counts as a hit if it lies within a configurable, point-specific tolerance (in milliseconds) of the reference point.

### Localization Accuracy

- **Mean Absolute Timing Error (MATE)**, i.e. the mean absolute error in milliseconds
- Signed timing error (mean and standard deviation) and RMSE

Localization accuracy measures the temporal difference between detected fiducial points and their matching reference annotations. Evaluation is performed separately for each fiducial point wherever a reference is available.

### Robustness Analysis

Additional experiments investigate how performance changes under:

- different noise types, noise levels and combined noise;
- variations in ECG morphology;
- different preprocessing configurations;
- different signal quality levels;
- fixed versus adaptive detection strategies.

Quantitative results, summary tables and visualizations are stored in the `results/` directory.

## Evaluation Protocol

The evaluation is designed to ensure fair and reproducible comparisons.

- Methods are evaluated under documented and consistent preprocessing conditions.
- Sampling frequencies and annotation locations are handled consistently; timing errors are reported in milliseconds.
- Reference annotations and predicted fiducial points are stored separately.
- Machine learning experiments use record-wise or subject-wise data separation, as appropriate for the dataset, to prevent data leakage. Parameter tuning uses training and validation data only.
- Performance is reported separately for each relevant fiducial point and dataset.
- The unit of statistical analysis is the record, because heartbeats from the same record are not independent samples. Confidence intervals use record-level resampling; method comparisons use nonparametric paired tests with multiple-comparison correction and effect sizes.
- Experimental configurations, software versions and random seeds are documented to support reproducibility.

## Limitations

- The framework is for research use only; it is not a medical device.
- The annotation rules of the databases differ, so comparisons are limited to jointly evaluable points.
- Reference annotations are not error-free; tolerance windows account for this.
- LUDB records are short (10 s), which limits statistical power for rare conditions and small subgroups.
- Synthetic signals approximate real physiological and measurement conditions only.
- The advantage of adaptive methods must be demonstrated by controlled experiments.
- Real-time processing is not a goal.

## Documentation

Requirements, use cases and design principles are described in the software requirements specification in `docs/`.

## References

1. Pan, J., & Tompkins, W. J. (1985). A Real-Time QRS Detection Algorithm. *IEEE Transactions on Biomedical Engineering*, 32(3), 230–236.
2. Imtiaz, M. N., & Khan, N. (2022). Pan-Tompkins++: A Robust Approach to Detect R-peaks in ECG Signals. arXiv:2211.03171.
3. Martínez, J. P., Almeida, R., Olmos, S., Rocha, A. P., & Laguna, P. (2004). A Wavelet-Based ECG Delineator: Evaluation on Standard Databases. *IEEE Transactions on Biomedical Engineering*, 51(4), 570–581.
4. Moskalenko, V., Zolotykh, N., & Osipov, G. (2019). Deep Learning for ECG Segmentation. arXiv:2001.04689.
5. Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. *MICCAI 2015*, LNCS 9351, 234–241.
6. Ahmed, A. A., Ali, W., Abdullah, T. A. A., & Malebary, S. J. (2023). Classifying Cardiac Arrhythmia from ECG Signal Using 1D CNN Deep Learning Model. Mathematics, 11(3), 562.
7. Moody, G. B., & Mark, R. G. (2001). The Impact of the MIT-BIH Arrhythmia Database. *IEEE Engineering in Medicine and Biology Magazine*, 20(3), 45–50.
8. Kalyakulina, A., et al. (2020). LUDB: A New Open-Access Validation Tool for Electrocardiogram Delineation Algorithms. *IEEE Access*, 8, 186181–186190.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Author 

Görözdi Attila. Developed as part of a Bachelor's thesis in Computer Science.

Széchenyi István University, Győr

2026-27