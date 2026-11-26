# ACLF_analysis
# ACLF mortality prediction analysis

The R script reproduces the analyses reported in the manuscript and supplement from authorized, analysis-ready cohorts. It assumes that study eligibility, selection of one hospitalization per patient, outcome coding, and first-day measurement selection have already been applied. Figure 1 uses aggregate flow counts supplied with the prepared input.

## Run the analysis

Use R 4.6.1 and a fresh output directory. Run the stages in this order.

```sh
Rscript --vanilla ACLF_analysis.R six prepared_input.rds output
Rscript --vanilla ACLF_analysis.R reduce prepared_input.rds output
Rscript --vanilla ACLF_analysis.R evaluate prepared_input.rds output
Rscript --vanilla ACLF_analysis.R singleton prepared_input.rds output
```

The `all` command runs these stages in separate R processes and requires each process to exit successfully.

| Stage | Reported analyses |
| --- | --- |
| `six` | Six model families, assessed by nested out-of-fold prediction in MIMIC (Figure 2A and Supplementary Table S3). |
| `reduce` | Predictor ranking and reduction for ElasticNet and LightGBM, final model fitting, and selection of the locked research cutoffs (Figure 2B–C and Table 2). |
| `evaluate` | Cohort characteristics, missingness, primary model performance, paired model comparisons, MELD and MELD-Na comparisons, and the Chinese sensitivity analyses (Figures 1–4, Tables 1–3, Supplementary Figures S1–S7, and Supplementary Tables S1, S2, and S5). |
| `singleton` | Comparison of batch and single-patient scoring in both external cohorts (Supplementary Figure S8 and Supplementary Table S4). |

The `reduce` stage uses `six_model_oof.rds` written by `six`. The `evaluate` stage uses `reduced_fit.rds` and `six_model_ranking.csv` from the preceding stages. The `singleton` stage uses `output/结果/核对/evaluation.rds` from `evaluate`. Report figures and aggregate tables are written under `output/结果`. The stage inputs, saved predictions, models, and checkpoints contain controlled-access analytical data and belong in the authorized local workspace.

## Prepared input

The RDS file must contain the following fields. Preserve the original development row order and align all outcomes, hospital groups, and supplemental vectors to their corresponding cohort tables.

| Field | Contents |
| --- | --- |
| `cohorts` | Named list in the order `MIMIC OOF`, `eICU external`, `China external`. Each entry supplies `x` (a predictor data frame with unique pseudonymous `patient_id`), `y` (numeric in-hospital death, 0/1), `kind` (`OOF` or `external`), and `cluster` (eICU hospital grouping or `NULL`). The Chinese cohort uses `x$data_source` to identify Mianyang (`四川队列`) and Nantong (`肝衰原`). Use the units described in the manuscript. |
| `training$candidates` | Ordered vector of the 43 initially profiled fields. |
| `training$availability` | Named missing fractions from the earlier eICU availability assessment, held fixed in development. |
| `reporting$profile` | Historical availability metadata with `variable`, `MIMIC_missing`, `eICU_missing`, `full_development_missing_le_50pct`, `external_missing_le_50pct`, and `both_cohorts_missing_le_50pct`. The displayed current eICU missingness is calculated from the prepared external cohort. |
| `reporting$ammonia_eicu` | Standardized eICU ammonia values aligned to those patients, used in descriptive reporting. |
| `reporting$flow_counts` | Aggregate MIMIC and eICU counts for Figure 1. |
| `variables` | Metadata with `variable`, `label`, and `unit` columns. |

The expected numbers of patients and in-hospital deaths are 1,618 and 405 in MIMIC, 293 and 99 in eICU, and 145 and 45 in China. Measured bilirubin, creatinine, INR, and sodium are needed for the matched MELD comparison.

## Software

The study used `glmnet` 5.0, `ranger` 0.18.0, `xgboost` 3.2.1.1, `lightgbm` 4.7.0, `catboost` 1.2.10, `kernlab` 0.9.33, `mice` 3.19.0, `pROC` 1.19.0.1, `jsonlite` 2.0.0, `digest` 0.6.39, `ggplot2` 4.0.3, and `scales` 1.4.0. The analysis uses local R workers. The paired-risk PDF uses Cairo.
