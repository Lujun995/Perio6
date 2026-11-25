# Perio6

Perio6 is a machine-learning approach to periodontitis surveillance using a partial-mouth examination. The R code uses clinical attachment loss (CAL) and probing pocket depth (PPD) measured at selected teeth to estimate moderate-or-severe and severe periodontitis. The generative classifier fits ordinal models for site-level measurements, imputes unobserved sites by Gibbs sampling, and applies periodontal case definitions to the completed measurements.

## Repository contents

| File | Purpose |
| --- | --- |
| [`src/case_def.R`](src/case_def.R) | Applies periodontal case definitions to site-level measurements. |
| [`src/fit.gen.R`](src/fit.gen.R) | Fits the ordinal models used to generate unobserved measurements. |
| [`src/predict.gen.R`](src/predict.gen.R) and [`src/predict.gen.dependencies.R`](src/predict.gen.dependencies.R) | Run Gibbs sampling, check sampled measurements, and return imputed data or predictions per participant. |
| [`src/prediction_models.R`](src/prediction_models.R) | Fits comparison models and evaluates partial-mouth examination protocols. Evaluation uses `calculate_metrics2.R` from the companion analysis repository. |
| [`examples/Model Fit.R`](examples/Model%20Fit.R) and [`examples/Model Predict.R`](examples/Model%20Predict.R) | Reference scripts for training and prediction. They use study-specific input files and paths. |

## Use the prediction functions

Install the core dependencies in R.

```r
install.packages(c("tidyverse", "ordinalNet"))
```

The comparison models also use `ranger`, `ROCR`, `glmnet`, and `e1071`.

Prepare a numeric matrix with one row per participant, unique participant IDs as row names, and the site columns expected by the fitted models. The NHANES models use names such as `ohx02lad` for CAL and `ohx02pcd` for PPD, with values coded from 0 to 6. Use `NA` for unmeasured sites. In the example, the six tooth numbers are `03`, `14`, `18`, `25`, `30`, and `31`.

From the repository root, replace the three file paths below with local paths to the fitted models and your prepared measurement matrix. The model RDS files are available under `res/` in the companion repository.

```r
source("src/case_def.R")
source("src/predict.gen.dependencies.R")
source("src/predict.gen.R")

cal_model <- readRDS("path/to/nhanes_cal_genmdl_ord.rds")
ppd_model <- readRDS("path/to/nhanes_ppd_genmdl_ord.rds")
measurements <- readRDS("path/to/measurements.rds")
teeth <- c("03", "14", "18", "25", "30", "31")
observed_sites <- colnames(measurements)[substr(colnames(measurements), 4, 5) %in% teeth]

predictions <- predict.gen(
  cal_mdl = cal_model,
  ppd_mdl = ppd_model,
  new_data = measurements,
  input_teeth = observed_sites,
  n_cores = 1,
  type = "results"
)
```

The results include `seqn`, `pred_MS` for moderate-or-severe periodontitis, `pred_sev_12` for severe periodontitis, and `n_val`, the number of retained draws. Join results by `seqn` for downstream analysis.

For fitting, `examples/Model Fit.R` expects `res/nhanes_complete.rds` and resolves paths from `examples/`. For prediction, `examples/Model Predict.R` requires a CSV path, paths to the source and model files, and the prepared numeric matrix as the prediction input. The function example above shows the input and calls needed for prediction from the repository root.

The study report, training artifacts, and analysis scripts are in [Perio6 Analyses and Results](https://github.com/Lujun995/Perio6_Analyses_and_Results).

## License

The code is released under the [MIT License](LICENSE). Cohort data follow the terms of their respective data providers.
