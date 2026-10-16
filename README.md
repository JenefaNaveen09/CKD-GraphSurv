# CKD-GraphSurv

**Knowledge-Guided Graph and Temporal Learning for Chronic Kidney Disease Risk Prediction Across Cohorts**

CKD-GraphSurv predicts time to CKD-related outcomes from longitudinal or single-visit laboratory data. It combines a knowledge-guided feature-graph transformer, a recurrent temporal encoder, trajectory channels, gated fusion, and survival heads. The model is developed on one cohort and tested on two external cohorts from different countries.

This repository holds the full Google Colab notebook used for the experiments in the accompanying manuscript.

---

## Highlights

- Feature-graph attention over 10 laboratory variables, guided by a clinical prior graph (e.g., eGFR–creatinine, calcium–phosphorus, albumin–proteinuria).
- GRU-based temporal encoder with positional encoding over 3-month bins.
- Discrete-time hazard output with a proportional-hazards head and a linear skip path.
- Missingness augmentation, so the same model can run on full histories or on a single visit.
- Internal, single-visit, and external evaluation with discrimination, prediction error, calibration, and decision-curve analysis.
- Ablations, paired bootstrap tests with Holm correction, Integrated Gradients, and attention analysis.

---

## Datasets

All data are public. The notebook downloads them automatically from Europe PMC supplementary files.

| Cohort | Country | Role | Patients | Source |
|---|---|---|---|---|
| Ajou | South Korea | Development and internal test | 2,255 | [10.1371/journal.pone.0204586](https://doi.org/10.1371/journal.pone.0204586) (CC BY 4.0) |
| CKD-ROUTE (renal) | Japan | External validation | 797 | [10.1186/s12882-021-02496-7](https://doi.org/10.1186/s12882-021-02496-7) |
| CKD-ROUTE (composite) | Japan | External validation | 1,138 | same as above |
| Foshan | China | External validation, 3-year outcome | 297 | same as above |

No raw patient data are stored in this repository. Please follow the licence terms of each original publication.

---

## Laboratory features

eGFR, creatinine, BUN, haemoglobin, albumin, bicarbonate, calcium, phosphorus, potassium, and proteinuria.

The external cohorts share only five of these (eGFR, creatinine, haemoglobin, albumin, proteinuria). The other features are masked during external inference.

---

## Models compared

| Type | Models |
|---|---|
| Proposed | CKD-GraphSurv |
| Deep survival | Temporal Transformer, GRU-DeepHit, DeepHit, DeepSurv |
| Conventional | CoxPH, Random Survival Forest (RSF) |

Each model is tuned with Optuna (60 trials) and trained with five seeds: 42, 7, 2026, 11, 99.

---

## Main results (from the manuscript)

| Setting | Metric | CKD-GraphSurv |
|---|---|---|
| Ajou internal test | C-index | 0.850 |
| Ajou single-visit test | C-index | 0.825 |
| CKD-ROUTE renal | C-index | 0.922 |
| CKD-ROUTE renal | IBS | 0.0749 (DeepHit 0.0984, RSF 0.0705) |
| Foshan | 3-year AUC | 0.842 |

The main gain is lower prediction error against selected deep baselines. It is not a consistent improvement on every metric. See the manuscript for the full tables and confidence intervals.

---

## Repository structure

```
CKD-GraphSurv/
├── CKD_GraphSurv_Final.ipynb   # Full pipeline (data → training → evaluation → figures)
├── requirements.txt
├── LICENSE
└── README.md
```

### Notebook sections

| Cell | Content |
|---|---|
| 0–1 | Install packages, imports, Drive mount, global settings |
| 2–4 | Download cohorts, harmonise variables, build sequences and the prior graph |
| 5 | Model definitions (CKD-GraphSurv and baselines) |
| 6 | Optuna hyperparameter search |
| 7–8 | Train proposed model and baselines across seeds |
| 9–10 | Ablations and stepwise build-up; prediction cache |
| 11–13 | C-index, time-dependent AUC, Brier score, IBS, calibration, decision curves |
| 14 | Paired bootstrap, Holm correction, non-inferiority test |
| 15–17 | Risk stratification, ablation summary, subgroup analysis |
| 18–20 | Integrated Gradients, attention maps, patient-level case studies |
| 21 | Statistical tables and export |

---

## How to run

1. Open `CKD_GraphSurv_Final.ipynb` in Google Colab.
2. Select a GPU runtime (Runtime → Change runtime type → T4 GPU).
3. Run all cells in order. Google Drive will be mounted for caching.

Outputs are written to:

```
/content/drive/MyDrive/CKD_GraphSurv_Public/
├── raw/              # downloaded CSV files
├── interim/          # Optuna database, predictions, bootstrap results
├── checkpoints_v3/   # trained model weights
├── figures_v3/       # all figures
└── results_v3/       # all result tables
```

Cached files are reused. A rerun after a disconnect resumes from the last saved step.

### Key settings

| Parameter | Value |
|---|---|
| Time bin | 3 months |
| Maximum horizon | 120 months |
| Evaluation horizons | 12, 24, 36 months |
| Calibration horizon | 24 months |
| C-index truncation (τ) | 36 months |

---

## Requirements

```
torch
numpy
pandas
scikit-learn
scikit-survival
lifelines
captum
optuna
matplotlib
seaborn
networkx
scipy
requests
```

Colab already includes most of these. The first cell installs the rest:

```bash
pip install scikit-survival lifelines captum optuna
```

---

## Citation

If you use this code, please cite the manuscript:

```bibtex
@article{ckdgraphsurv,
  title   = {CKD-GraphSurv: Knowledge-Guided Graph and Temporal Learning for Chronic Kidney Disease Risk Prediction Across Cohorts},
  author  = {Author(s)},
  journal = {Under review},
  year    = {2026}
}
```

Please also cite the original dataset publications listed above.

---

## Disclaimer

This code is for research use only. It is not a validated clinical tool and should not guide patient care.

## License

Released under the MIT License. See `LICENSE` for details.
