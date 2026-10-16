# CKD-GraphSurv

**Knowledge-Guided Graph and Temporal Learning for Chronic Kidney Disease Risk Prediction Across Cohorts**

CKD-GraphSurv is a research framework for predicting chronic kidney disease (CKD)-related outcomes from longitudinal or single-visit clinical observations. It combines a knowledge-guided feature-graph transformer, recurrent temporal encoding, trajectory channels, gated fusion, and survival prediction heads. Evaluation covers discrimination, prediction error, calibration, risk stratification, and model explanations across three cohorts.

## Repository status

This README documents the accompanying CKD-GraphSurv manuscript. A runnable CKD-GraphSurv implementation, dependency specification, trained checkpoints, and processed datasets are not included in the supplied materials. Installation and execution commands should be added when the corresponding implementation is released.

**File compatibility:** `TokenGaussFormer_NIDS_Final.ipynb` implements TokenGauss-Former for open-set IoT intrusion detection on CICIoT2023 and ToN-IoT. It is a separate project and does not reproduce the CKD-GraphSurv experiments. Keep that notebook in a separate repository.

## Framework

The processing sequence described in the manuscript is:

1. Construct longitudinal observation windows or single-visit inputs.
2. Encode laboratory availability and static clinical characteristics.
3. Preprocess measurements and derive trajectory channels.
4. Learn relationships among clinical variables using a knowledge-guided feature-graph transformer.
5. Encode temporal observations with a gated recurrent unit (GRU) and aggregate the resulting representations.
6. Combine complementary representations through gated fusion.
7. Generate survival and cumulative-risk predictions using the GRU hazard core, proportional-hazards head, and linear skip path.
8. Apply internally fitted recalibration and assign risk groups using development-derived thresholds.
9. Examine predictions using integrated gradients and graph/temporal attention displays.

Cumulative event risk is defined as `R(t) = 1 − S(t)`, where `S(t)` is the predicted survival probability at time `t`.

## Clinical inputs

| Input type | Variables |
| --- | --- |
| Laboratory measurements | eGFR, creatinine, haemoglobin, albumin, proteinuria, blood urea nitrogen, bicarbonate, calcium, phosphorus, potassium |
| Static characteristics | Age, sex, diabetes, hypertension, glomerulonephritis |
| Availability representation | Observed/missing indicator for each laboratory variable and observation representation |
| Longitudinal information | Baseline, a time-weighted summary of the six-month run-in window, six-month measurements, and trajectory channels |
| Restricted-input setting | Single-visit observations; external cohorts provide five shared laboratory variables and five static variables |

The middle longitudinal representation is a time-weighted window summary, rather than an independent three-month visit. Preserve cohort-specific units, endpoint definitions, and measurement availability when preparing data.

## Cohorts and evaluation settings

| Cohort / analysis | Patients | Events | Role | Outcome |
| --- | ---: | ---: | --- | --- |
| Ajou, South Korea | 2,255 | 708 | Development and internal testing | Renal replacement therapy |
| CKD-ROUTE renal, Japan | 797 | 185 | External validation | Kidney failure or ≥50% eGFR decline |
| CKD-ROUTE composite, Japan | 1,138 | 280 | Sensitivity analysis | Kidney events, cardiovascular events, or death |
| Foshan, China | 297 | 108 | External validation | Composite outcome within three years |

Ajou is divided into **1,578 training**, **226 validation**, and **451 test** patients. The single-visit assessment uses the same 451 test patients. These two Ajou settings are paired evaluations, not independent cohorts. CKD-ROUTE renal and composite analyses also concern different analysis populations and endpoints and should be interpreted separately.

The manuscript identifies Ajou supplementary data from PLOS ONE (`pone.0204586`) and CKD-ROUTE/Foshan supplementary data from BMC Nephrology (2021). Consult the manuscript references and original data providers for exact files, access conditions, and permitted reuse. Clinical datasets are not bundled here.

## Training configuration reported in the manuscript

| Setting | Value |
| --- | --- |
| Hyperparameter search | 60 tree-structured Parzen estimator trials per model |
| Pruning | Median pruning |
| Selected learning rate | 0.00133 |
| Selected weight decay | 4.42 × 10⁻⁵ |
| Batch size | 128 |
| Dropout | 0.008 |
| Ranking-loss weight | 0.05 |
| Reconstruction-loss weight | 0.5 |
| Random seeds | 42, 7, 2026, 11, 99 |
| Checkpoint selection | Validation performance under full-window and single-visit inputs |
| Recalibration | Fitted on internal validation predictions and outcomes; fixed during evaluation |
| Risk-group thresholds | Development-derived 50th and 85th percentiles of predicted three-year risk |

These values describe the manuscript experiments; they are not a substitute for the implementation or an environment lockfile.

## Reported results

The following are manuscript-reported mean estimates. They have not been independently reproduced from the supplied notebook.

| Evaluation setting | Discrimination measure | Value | Prediction-error measure | Value |
| --- | --- | ---: | --- | ---: |
| Ajou internal test | C-index | 0.850 | Integrated Brier score | 0.0857 |
| Ajou single-visit test | C-index | 0.825 | Integrated Brier score | 0.1033 |
| CKD-ROUTE renal | C-index | 0.922 | Integrated Brier score | 0.0749 |
| Foshan three-year outcome | ROC AUC | 0.842 | Brier score | 0.1669 |

Higher C-index/AUC and lower Brier scores indicate better performance within the corresponding evaluation setting. A three-year binary-outcome AUC is distinct from a survival C-index, and a fixed-horizon Brier score is distinct from an integrated Brier score. Cross-cohort values also reflect differences in endpoints, available inputs, and patient populations.

The findings support competitive discrimination and lower prediction error than selected deep survival baselines. They do not establish an advantage over every comparator: CoxPH has a slightly lower Ajou internal IBS (0.0853), and random survival forest has a lower CKD-ROUTE renal IBS (0.0705). Foshan calibration remains a limitation, with a reported observed-to-expected ratio of 1.380.

## Comparison methods and analyses

The manuscript compares CKD-GraphSurv with:

- Temporal Transformer
- GRU-DeepHit
- DeepHit
- DeepSurv
- Random survival forest (RSF)
- Cox proportional hazards (CoxPH)

Evaluation includes time-dependent AUC, survival concordance, Brier scores, calibration, decision-curve analysis, paired-bootstrap comparisons, component ablations, and risk stratification. Interpretation uses integrated gradients alongside graph and temporal attention. These explanations describe model behaviour and do not establish causal relationships.

## Citation

**CKD-GraphSurv: Knowledge-Guided Graph and Temporal Learning for Chronic Kidney Disease Risk Prediction Across Cohorts.**
