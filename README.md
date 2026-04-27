# Forecasting

## Zillow Sales House Forecasting

End-to-end hierarchical forecasting project built from Zillow metropolitan sales-count series. The project covers data preprocessing, exploratory analysis, spatial analysis, statistical benchmarks, machine learning benchmarks, deep learning models, cross-sectional reconciliation, and post-hoc error analysis.

## Project Overview

The dataset was built bottom-up from Zillow metropolitan sales-count series. Metropolitan areas define the base level. Missing monthly values were filled with zero after aligning all series to a common monthly calendar.

State-level series were obtained by summing metropolitan areas within each state group, and the national series was obtained by summing the state-level aggregates. Therefore, the cleaned national series represents the sum of the included metropolitan areas, not Zillow’s original national aggregate. This design ensures exact cross-sectional coherence for hierarchical forecasting and reconciliation.

The final dataset follows a coherent three-level hierarchy:

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

The spatial analysis uses latitude and longitude to explore total sales and average monthly sales by region. Geohash and H3 spatial indexing were used to summarize geographic market concentration.

The results show clear spatial concentration patterns, especially around New York and the state of Florida.

Interactive H3 maps are available here:

- [Combined H3 spatial map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_combined_map.html)
- [H3 market density map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_market_density.html)
- [H3 sales concentration map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_sales_concentration.html)

## Forecasting Design

Forecasting was evaluated using walk-forward cross-validation with:

- Forecast horizon: 12 months
- Number of cutoffs: 5
- Step size: 12 months

For each model and cutoff, in-sample forecasts were generated to compute residuals for reconciliation, while out-of-sample forecasts were used for final evaluation and reconciled forecasts.

## Benchmark Models

### Statistical Benchmarks

Implemented in `statistical_benchmarks_crossvalidation`. The statistical models were estimated using `StatsForecast`:

- SeasonalNaive as the reference benchmark
- Holt-Winters
- TBATS
- AutoARIMA
- AutoARIMAX with monthly calendar regressors

### Machine Learning Benchmarks

Implemented in `ML_benchmarks_crossvalidation`. Global models were trained using:

- LightGBM
- CatBoost
- XGBoost

Autocorrelation diagnostics and partial autocorrelation plots were inspected to define relevant lag structure, with special attention to 12-month seasonal lags. The models used lag features, lag transformations, and basic calendar features such as month, quarter, and year.

### Deep Learning Benchmarks

Implemented in `DL_benchmarks_crossvalidation`. The deep learning benchmark included:

- GRU
- NHITS
- NBEATSx
- KAN

The models incorporated historical calendar variables and were configured with 200 training epochs, a learning rate of 0.0001, and MAE loss. Additional model-specific details are included in the notebook.

## Hierarchical Reconciliation

Implemented in `hierarchical_reconciliation`.

Cross-sectional reconciliation was applied in R using the `FoReco` package over the same three-level hierarchy defined above.

For each model and cutoff:

- In-sample forecasts were used to compute residuals.
- Out-of-sample forecasts were reconciled.

The reconciliation methods included bottom-up, top-down, middle-out, least-squares, weighted least-squares, shrinkage covariance-based reconciliation, and level-conditional coherent reconciliation.

## Error Analysis

Implemented in `error_analysis`.

NRMSE was selected as the main metric because it allows comparison across hierarchy levels with different sales magnitudes. The analysis includes global model ranking, reconciliation improvements, comparison against SeasonalNaive, bias factor, and error distributions.

![Global error comparison](Zillow%20Sales%20House/Figures/global_error.png)

![Reconciliation improvement](Zillow%20Sales%20House/Figures/Improvement reconciliation.png)

## Best Global Results

Lower NRMSE is better.

| Model | Reconciliation method | NRMSE |
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

The best global performance was obtained by Holt-Winters with LCC-SHR reconciliation, followed closely by AutoARIMAX with bottom-up reconciliation and TBATS with LCC-SHR. Several reconciled models improved over the SeasonalNaive benchmark, confirming the value of hierarchical reconciliation for this dataset.

However, reconciliation did not improve all models equally. Some methods were more effective for statistical models, while several machine learning and deep learning models showed model-dependent gains. This highlights the importance of evaluating both the base model and the reconciliation method jointly.

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
│       ├── eda_national_sales.png
│       ├── hierarchy_series.png
│       ├── global_error.png
│       ├── reconciliation_improvement.png
│       └── best_reconciled_models.png
├── README.md
└── LICENSE
```

## Notes

Raw datasets and large intermediate files are not included in this repository. The notebooks document the full workflow and can be adapted to similar hierarchical forecasting problems.
