# Forecasting

Portfolio of forecasting projects focused on hierarchical, spatial, and time series modeling.

## Zillow Home Sales

End-to-end hierarchical forecasting project built from Zillow metropolitan sales-count series. The project covers data preprocessing, exploratory analysis, spatial analysis, statistical benchmarks, machine learning benchmarks, deep learning models, cross-sectional reconciliation, and error analysis.

## Project Overview

The dataset was built bottom-up from publicly available Zillow metropolitan sales-count series. Metropolitan areas define the base level. Missing monthly values were filled with zero after aligning all series to a common monthly calendar.

State-level series were obtained by summing metropolitan areas within each state group, and the national series was obtained by summing the state-level aggregates. The cleaned national series therefore represents the sum of the included metropolitan areas, not Zillow’s original national aggregate. This guarantees exact cross-sectional coherence for hierarchical forecasting and reconciliation.

The final dataset follows a three-level hierarchy:

```text
Country → State → Region
```

It contains:

- 1 national series
- 49 state-level series
- 300 regional series

National sales volume peaked in 2021, with approximately 4.99 million sales, followed by a clear contraction in 2022 and 2023. Values for 2026 should be interpreted cautiously because the year is incomplete.

![National home sales by year](Zillow%20Sales%20House/Figures/EDA_zillow.png)

## Exploratory Data Analysis

At the state level, Florida, California, Texas, New York, and Pennsylvania show the highest total sales volume. At the regional level, the largest markets are New York, NY; Chicago, IL; Miami, FL; Los Angeles, CA; and Atlanta, GA.

The largest region accounts for about 4.6% of total regional sales, the top 10 regions account for about 28.1%, and the top 20 account for about 42.9%.

![Hierarchical sales series](Zillow%20Sales%20House/Figures/hierarchhy.png)

## Spatial Analysis

Latitude and longitude were used to explore total sales and average monthly sales by region. Geohash and H3 spatial indexing were applied to summarize geographic market concentration.

The results show clear spatial concentration patterns, especially around New York and Florida.

Interactive H3 maps:

- [Combined H3 spatial map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_combined_map.html)
- [H3 market density map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_market_density.html)
- [H3 sales concentration map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_sales_concentration.html)

## Forecasting Design

Forecasting was evaluated with walk-forward cross-validation:

- Forecast horizon: 12 months
- Number of cutoffs: 5
- Step size: 12 months

For each model and cutoff, in-sample forecasts were generated to compute residuals for reconciliation. Out-of-sample forecasts were used for final evaluation.

## Benchmark Models

### Statistical Benchmarks

Implemented in statistical_benchmarks_crossvalidation. The statistical models were estimated using StatsForecast:

Models:

- SeasonalNaive
- Holt-Winters
- TBATS
- AutoARIMA
- AutoARIMAX with monthly calendar regressors

### Machine Learning Benchmarks

Implemented in `ML_benchmarks_crossvalidation`.

Global models:

- LightGBM
- CatBoost
- XGBoost

Partial autocorrelation diagnostics were used to inspect lag structure, with special attention to 12-month seasonal lags. The models used lag features, lag transformations, and calendar features such as month, quarter, and year.

### Deep Learning Benchmarks

Implemented in `DL_benchmarks_crossvalidation`.

Models:

- GRU
- NHITS
- NBEATSx
- KAN

The models incorporated historical calendar variables and were configured with 200 training epochs, a learning rate of 0.0001, and MAE loss. Additional model-specific details are included in the notebook.

## Hierarchical Reconciliation

Implemented in `hierarchical_reconciliation`.

Cross-sectional reconciliation was applied in R using the `FoReco` package over the same three-level hierarchy. In-sample residuals were used to estimate reconciliation weights, and out-of-sample forecasts were reconciled.

The reconciliation methods included bottom-up, top-down, middle-out, least-squares, weighted least-squares, shrinkage covariance-based reconciliation, and level-conditional coherent reconciliation.

## Error Analysis

Implemented in `error_analysis`.

NRMSE was selected as the main metric because it allows comparison across hierarchy levels with different sales magnitudes. The analysis includes global ranking, reconciliation improvements, comparison against SeasonalNaive, bias factor, and error distributions.

![Global error comparison](Zillow%20Sales%20House/Figures/global_error.png)

![Reconciliation improvement](Zillow%20Sales%20House/Figures/Improvement%20reconciliation.png)

## Best Global Results

Lower NRMSE is better.

| Model | Reconciliation Method | NRMSE |
|---|---:|---:|
| HoltWinters | LCC-SHR | 0.1439 |
| AutoARIMAX | BU | 0.1451 |
| TBATS | LCC-SHR | 0.1469 |
| MFLES | SHR | 0.1572 |
| NHITS | BU | 0.1593 |
| NBEATSx | SHR | 0.1643 |
| AutoARIMA | LCC-SHR | 0.1682 |
| CatBoost | LCC-SHR | 0.1731 |
| LightGBM | TD-GSF | 0.1800 |
| XGBoost | LCC-SHR | 0.1846 |
| KAN | BU | 0.1985 |
| SeasonalNaive | Base | 0.2022 |
| GRU | BU-SNTZ | 0.2355 |

![Best reconciled models](Zillow%20Sales%20House/Figures/best_models_reconcilied.png)

## Main Findings

Holt-Winters with LCC-SHR achieved the best global NRMSE, followed closely by AutoARIMAX with Bottom-Up reconciliation and TBATS with LCC-SHR. Several reconciled models improved over the SeasonalNaive benchmark, showing the value of hierarchical reconciliation for this dataset.

However, reconciliation did not improve all models equally. Some methods were more effective for statistical models, while several machine learning and deep learning models showed model-dependent gains. This highlights the importance of evaluating both the base model and the reconciliation method jointly.

## My Publications and Research Work

I am an author or co-author of the following forecasting and time-series research works.

| Work | Link | Scope | Main Finding |
|---|---|---|---|
| Temporal hierarchical forecast reconciliation of photovoltaic power generation from heterogeneous base models | [Paper](https://doi.org/10.1016/j.meaene.2026.100094) | Temporal reconciliation for Belgian photovoltaic generation across weekly, daily, and hourly resolutions. | LightGBM was the strongest baseline; cross-covariance reconciliation achieved the best overall performance, with average error reductions of approximately 15% across frequencies. |
| Short-Term Hierarchical Photovoltaic Forecasting with Cross-Sectional Reconciliation in Belgium | [Preprint](https://doi.org/10.2139/ssrn.5737222) | Short-term photovoltaic forecasting across national, regional, and provincial levels in Belgium. | LightGBM with Bottom-Up reconciliation achieved the lowest NRMSE; NHITS and NBEATSx showed the largest relative error reductions. |
| Heuristic Cross-Temporal Reconciliation Applied to Heterogeneous Models in Photovoltaic Forecasting | [Preprint](https://doi.org/10.2139/ssrn.5527782) | Cross-temporal reconciliation combining spatial and temporal hierarchies for photovoltaic forecasting. | Deep learning models benefited the most; KAN with iterative reconciliation achieved the lowest global error. |
| Sequential prediction of pediatric glucose dynamics using LSTM Networks Trained on Synthetic Physiological Data | [Paper](https://doi.org/10.35429/JIT.2025.12.32.2.1.11) | LSTM-based glucose prediction for pediatric Type 1 Diabetes using synthetic physiological data. | The model captured simulated glycemic dynamics with R² = 0.93, MAE = 4.52 mg/dL, and RMSE = 5.66 mg/dL. |

## Repository Structure

```text
Forecasting/
├── Zillow Sales House/
│   ├── Data Preprocessing EDA/
│   ├── Spatial Analysis/
│   │   └── Outputs/
│   │       ├── zillow_h3_combined_map.html
│   │       ├── zillow_h3_market_density.html
│   │       └── zillow_h3_sales_concentration.html
│   ├── Statistical Benchmarks Crossvalidation/
│   ├── ML Benchmarks Crossvalidation/
│   ├── DL Benchmarks Crossvalidation/
│   ├── Hierarchical Reconciliation/
│   ├── Error Analysis/
│   └── Figures/
│       ├── EDA_zillow.png
│       ├── hierarchhy.png
│       ├── global_error.png
│       ├── Improvement reconciliation.png
│       └── best_models_reconcilied.png
├── README.md
└── LICENSE
```

## Notes

Raw datasets and large intermediate files are not included in this repository. The notebooks document the full workflow and can be adapted to similar hierarchical forecasting problems.
