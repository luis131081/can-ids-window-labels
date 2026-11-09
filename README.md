# Window-Level Labels Reveal Sparse-Injection Failures in CAN Intrusion Detection

Code and results for the article *"Window-Level Labels Reveal Sparse-Injection Failures in Controller Area Network Intrusion Detection: Evaluation of a Compact Convolutional–Recurrent Detector"* (L. Sandoval and L. Rivera, submitted to *IEEE Access*).

The repository contains the notebooks used to:

- reconstruct **window-level labels** from injected frames in OTIDS and Car-Hacking;
- compare **file-level vs. window-level** targets under **random vs. chronological** partitions;
- evaluate trivial baselines, tuned tree ensembles, logistic regression and a compact **CNN–LSTM** with an identifier embedding;
- run the ablation, window-length, injection-density, latency and statistical analyses (exact McNemar with Holm correction, bootstrap confidence intervals).

## Datasets (not redistributed)

Both datasets belong to the Hacking and Countermeasure Research Lab (HCRL) and must be downloaded from the original source after registration:

| Dataset | Source | Files used |
| --- | --- | --- |
| OTIDS (CAN intrusion dataset) | https://ocslab.hksecurity.net/Dataset/CAN-intrusion-dataset | `Attack_free_dataset.txt`, `DoS_attack_dataset.txt`, `Fuzzy_attack_dataset.txt`, `Impersonation_attack_dataset.txt` |
| Car-Hacking | https://ocslab.hksecurity.net/Datasets/car-hacking-dataset | `DoS_dataset.csv`, `Fuzzy_dataset.csv`, `gear_dataset.csv`, `RPM_dataset.csv` |

The notebooks expect the following layout in Google Drive (edit the paths at the top of each notebook if you use another location):

```
MyDrive/
└── Datasets/
    ├── Attack_free_dataset.txt
    ├── DoS_attack_dataset.txt
    ├── Fuzzy_attack_dataset.txt
    ├── Impersonation_attack_dataset.txt
    └── CarHacking/
        ├── DoS_dataset.csv
        ├── Fuzzy_dataset.csv
        ├── gear_dataset.csv
        └── RPM_dataset.csv
```

Results are written to `MyDrive/CAN_IDS_FINAL_EVIDENCE/REVISION` (OTIDS) and `MyDrive/CAN_IDS_FINAL_EVIDENCE/CARHACKING` (Car-Hacking).

## Repository structure

```
notebooks/
  modelo_2026V2.ipynb                   OTIDS parser, validation, random partition, initial CNN-LSTM (V2)
  modelo_2026V3.ipynb                   Architecture selection among candidates A, B and C (V3-C selected)
  modelo_2026_temporal.ipynb            Chronological hold-out with V3-C; random vs. chronological comparison
  revision_experimentos_CAN_IDS.ipynb   Main OTIDS experiments (blocks R0-R11)
  carhacking_FINAL.ipynb                Car-Hacking experiments (blocks CH1-CH3, summary, density)
  bootstrap_CI.ipynb                    95% bootstrap confidence intervals from saved predictions
results/
  otids/                                CSV outputs of revision_experimentos_CAN_IDS.ipynb
  carhacking/                           CSV outputs of carhacking_FINAL.ipynb
figures/                                Confusion-matrix figures (PDF)
```

Note: rows starting with "CH_" in `results/otids/revision_results.csv` come from a preliminary one-seed Car-Hacking test; the final Car-Hacking results are in `results/carhacking/`.

## How to reproduce

All experiments were run in Google Colab. A GPU (T4) is recommended; every notebook also runs on CPU, more slowly.

1. Download the datasets and place them as shown above.
2. **OTIDS** – open `notebooks/revision_experimentos_CAN_IDS.ipynb` and run the cells in order:
   - Section 0 (parser, validation, chronological split, windows) and **R0** (configuration; `QUICK = False` for the full run).
   - **R1–R3**: identifier field diagnosis (remote-frame flag), injected-frame rules and window-level labels.
   - **R4**: trivial baselines. **R5**: file- vs. window-level target × random vs. chronological partition, five seeds.
   - **R6**: tuned baselines (grid search on the chronological validation split). **R7**: ablation and identifier embedding. **R8**: window length.
   - **R9**: summary (mean ± SD), McNemar with Holm correction. **R10**: latency and model size. **R11**: figures.
   - Results are saved after every model and seed; set `RESUME = True` in R0 to continue an interrupted run.
3. **Car-Hacking** – open `notebooks/carhacking_FINAL.ipynb` and select *Run all*. It uses the first 1,000,000 frames of each capture and three seeds (about 30–40 min on a T4 GPU).
4. **Confidence intervals** – run `notebooks/bootstrap_CI.ipynb`; it reads the saved predictions of steps 2 and 3.

The notebooks `modelo_2026V2.ipynb`, `modelo_2026V3.ipynb` and `modelo_2026_temporal.ipynb` document the development of the original submission (architecture selection and the first random vs. chronological comparison) and are provided for traceability.

## Labeling rules (window-level target)

| Dataset | Rule for an injected frame |
| --- | --- |
| OTIDS – DoS | identifier `0x000` |
| OTIDS – Impersonation | identifier `0x164` with payload `01 02 03 04 05 06 07 08` |
| OTIDS – Fuzzy | not identifiable (spoofs legitimate identifiers); file-level label retained |
| Car-Hacking – all attacks | frame flag `T` provided by the dataset |

A window is labeled with attack class *c* if it contains at least one injected frame of scenario *c*; otherwise it is labeled attack-free/Normal.

## Main settings

- Windows of 20 consecutive frames, non-overlapping, built within each capture and each partition.
- Chronological partition: first 64% / next 16% / last 20% of each capture's frames, applied before windowing.
- Features: identifier, DLC and eight payload bytes (plus log inter-arrival time and identifier embedding in the final model); min–max scaling fitted on training data only. The remote-frame flag is **not** used as a feature (it leaks the Impersonation label).
- Seeds: 42, 7, 123, 2024, 31415 (OTIDS); 42, 7, 123 (Car-Hacking).
- Model selection by validation macro-F1 only; the test set is never used for selection.

## Software

Python 3.13, TensorFlow 2.20.0 (Keras 3.13), scikit-learn 1.6.1, NumPy, pandas, SciPy, Matplotlib. See `requirements.txt`.

Results obtained on GPU are not bit-for-bit deterministic; we therefore report means and standard deviations over seeds.

## Citation

If you use this code, please cite the article (citation will be added upon publication) and the archived release:

> L. Sandoval and L. Rivera, "Code and results for window-level evaluation of CAN intrusion detection," Zenodo, 2026, doi: [ADD DOI].

## License

Code: MIT License (see `LICENSE`). The datasets are subject to the terms of their original providers.
