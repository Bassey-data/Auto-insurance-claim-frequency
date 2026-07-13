# Predicting Auto Insurance Claim Frequency: Poisson GLM vs LightGBM

A claim frequency modeling project on the freMTPL2freq dataset, a real-world French motor third-party liability insurance portfolio of 678,013 policies. The project compares a Poisson GLM (the actuarial industry standard) against a LightGBM gradient boosting model, with a focus on the modeling decisions that actually matter for frequency data: exposure offsets, Poisson deviance, and Lorenz curve discrimination rather than accuracy or RMSE.

## Results

| Model | Test Deviance | vs Naive Baseline | Gini Index |
|---|---|---|---|
| Naive baseline | 0.3294 | 0.0% | 0.000 |
| Poisson GLM | 0.3191 | -3.1% | -0.047 |
| LightGBM | 0.3016 | -8.4% | 0.101 |

LightGBM's advantage comes from capturing VehAge's non-linear relationship with claim frequency. The GLM's single linear coefficient for VehAge (-0.041) cannot represent a relationship that requires multiple decision tree splits. This shows up clearly in the feature importance comparison: VehAge is the second most important feature in LightGBM but does not appear in the GLM's top 10.

The GLM's negative Gini (-0.047) and S-shaped Lorenz curve are worth noting. Despite statistically significant coefficients and a 3.1% deviance improvement, the GLM's risk ranking is backwards for the lowest-risk 30% of the portfolio. Using it for pricing tier construction would create adverse selection risk.

## Dataset

freMTPL2freq: French motor third-party liability insurance, frequency component. 678,013 policies with 12 features including driver age, vehicle age, bonus-malus score, region, and exposure.

The dataset is not committed to this repo (36MB). Download from [OpenML](https://www.openml.org/d/41214) and place in `data/raw/`, then run:

    python src/load_data.py

## Project structure

    notebooks/
        01_data_prep.ipynb           # Custom ARFF parser, dtype conversion, parquet
        02_eda.ipynb                 # Portfolio frequency, exposure-weighted analysis
        03_feature_engineering.ipynb # Outlier capping, encoding, train/test split
        04_glm_modeling.ipynb        # Poisson GLM, coefficients, deviance, Lorenz curve
        05_gbm_modeling.ipynb        # LightGBM with Poisson objective and exposure offset
        06_comparison.ipynb          # Side-by-side metrics, overlaid Lorenz curves
    reports/
        glm_results.json
        gbm_results.json
        Insurance_Claim_Frequency_Report.docx

## Key modeling decisions

**Exposure as an offset, not a feature.** A policy active for 6 months has half the opportunity to generate a claim as one active for 12 months. Both models encode this via a log(Exposure) offset with a fixed coefficient of 1.

**Poisson deviance, not RMSE.** Under a Poisson distribution, variance equals the mean. RMSE treats a prediction error of 0.1 at a true rate of 0.05 the same as the same error at a true rate of 5.0. Poisson deviance weights errors correctly.

**LightGBM offset handling.** LightGBM does not automatically reapply init_score at prediction time. Raw scores must be extracted and the offset added back manually: exp(raw_score + log(exposure)). Skipping this step silently invalidates all predictions.

**Encoding strategy.** Area is encoded as an ordinal integer (A=0 through F=5) to preserve its monotonic gradient. Nominal categoricals use one-hot encoding for the GLM and native category dtype for LightGBM.

## Setup

    pip install pandas numpy scikit-learn statsmodels lightgbm matplotlib seaborn pyarrow

## Full report

A 16-page technical report covering methodology, coefficient interpretation, evaluation framework, and business implications is available in `reports/Insurance_Claim_Frequency_Report.docx`.
