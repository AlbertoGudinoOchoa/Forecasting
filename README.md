# Forecasting

Portfolio repository focused on time series forecasting, hierarchical modeling, spatial analysis, and forecast reconciliation.

## Zillow Home Sales Forecasting

End-to-end forecasting project using public Zillow monthly metropolitan sales-count data.

The project builds a coherent monthly hierarchy from regional housing-market series, evaluates statistical, machine learning, and deep learning models, applies cross-sectional reconciliation, and analyzes forecast error across aggregation levels.

```text
Country → State → Region
```

The final dataset contains:

- 1 national monthly series
- 49 state-level monthly series
- 300 regional monthly series

State and national series were reconstructed by summing the included regional series. The national series therefore represents the modeled metropolitan markets, not Zillow’s original national aggregate. This keeps the hierarchy coherent for reconciliation.

## Why This Project Matters

Real estate forecasts are often used at several levels at once: national, state, and regional. A model can perform well at the aggregate level while missing important regional behavior. This project focuses on that problem by combining hierarchical forecasting, spatial analysis, reconciliation, and model comparison under the same validation design.

The workflow covers:

- monthly panel data cleaning,
- hierarchical aggregation,
- spatial analysis with latitude, longitude, Geohash, and H3,
- statistical, ML, and deep learning benchmarks,
- walk-forward cross-validation,
- cross-sectional forecast reconciliation,
- error and bias analysis.

## Data and Exploratory Analysis

The dataset was built from public Zillow monthly metropolitan sales-count series. All series were aligned to a common monthly calendar.

National sales volume peaked in 2021 at approximately 4.99 million sales, followed by a clear contraction in 2022 and 2023. This makes the forecasting problem more than a simple seasonal task: the models must handle both monthly seasonality and a market shift after 2021.

Values for 2026 should be interpreted carefully because the year is incomplete.

![National home sales by year](Zillow%20Sales%20House/Figures/EDA_zillow.png)

At the state level, Florida, California, Texas, New York, and Pennsylvania show the highest total sales volume.

At the regional level, the largest markets are:

- New York, NY
- Chicago, IL
- Miami, FL
- Los Angeles, CA
- Atlanta, GA

The largest region accounts for about 4.6% of total regional sales. The top 10 regions account for about 28.1%, and the top 20 regions account for about 42.9%.

This concentration matters because national-level accuracy can hide regional errors.

![Hierarchical sales series](Zillow%20Sales%20House/Figures/hierarchhy.png)

## Spatial Analysis

Latitude and longitude were used to analyze geographic concentration in sales volume. Geohash and H3 indexing were used to summarize regional density and market concentration.

The spatial analysis shows that a limited number of large metropolitan areas, especially around New York and Florida, contribute a substantial share of total sales activity.

Interactive H3 maps:

- [Combined H3 spatial map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_combined_map.html)
- [H3 market density map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_market_density.html)
- [H3 sales concentration map](https://albertogudinoochoa.github.io/Forecasting/Zillow%20Sales%20House/Spatial%20Analysis/Outputs/zillow_h3_sales_concentration.html)

## Forecasting Design

Forecasts were evaluated with walk-forward cross-validation.

| Setting | Value |
|---|---:|
| Frequency | Monthly |
| Horizon | 12 months |
| Cutoffs | 5 |
| Step size | 12 months |

Out-of-sample forecasts were used for final evaluation. In-sample forecasts were used only to estimate residuals for reconciliation.

## Models

### Statistical Models

Implemented in `Statistical Benchmarks Crossvalidation`.

- SeasonalNaive
- Holt-Winters
- TBATS
- MFLES
- AutoARIMA
- AutoARIMAX with monthly calendar regressors

### Machine Learning Models

Implemented in `ML Benchmarks Crossvalidation`.

- LightGBM
- CatBoost
- XGBoost

The ML models used lag features, rolling transformations, and calendar features such as month, quarter, and year.

### Deep Learning Models

Implemented in `DL Benchmarks Crossvalidation`.

- GRU
- NHITS
- NBEATSx
- KAN

The deep learning models used historical calendar variables and MAE loss. Model-specific settings are documented in the notebooks.

## Hierarchical Reconciliation

Implemented in `Hierarchical Reconciliation`.

Cross-sectional reconciliation was applied in R using the `FoReco` package. The reconciliation methods included:

- Bottom-Up
- Top-Down
- Middle-Out
- Least Squares
- Weighted Least Squares
- Shrinkage covariance-based reconciliation
- Level-Conditional Coherent reconciliation

Reconciliation was evaluated as part of the modeling pipeline, not assumed to improve every model.

## Error Analysis

Implemented in `Error Analysis`.

NRMSE was used as the main metric because it supports comparison across hierarchy levels with different sales volumes.

The analysis includes:

- global model ranking,
- improvement from reconciliation,
- comparison against SeasonalNaive,
- bias factor,
- error distributions across models and methods.

![Global error comparison](Zillow%20Sales%20House/Figures/global_error.png)

![Reconciliation improvement](Zillow%20Sales%20House/Figures/Improvement%20reconciliation.png)

## Best Global Results

Lower NRMSE is better.

| Model | Reconciliation Method | NRMSE |
|---|---|---:|
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

Statistical models performed best on this monthly dataset. Holt-Winters, AutoARIMAX, and TBATS achieved the lowest global errors after reconciliation, suggesting that trend and seasonality explain much of the signal in monthly housing sales.

NHITS and NBEATSx were the most competitive deep learning models, but they did not clearly outperform the best statistical benchmarks in this setup.

Reconciliation improved several models, but not uniformly. The best results depended on the combination of base model and reconciliation method, so reconciliation should be selected empirically rather than applied by default.

The post-2021 market contraction affected the forecasting task. Models had to capture both recurring monthly patterns and a structural change in sales volume.

Spatial concentration is also relevant. A limited number of large metropolitan areas drives a large share of total volume, which makes level-wise error analysis necessary.

## Future Work

The current ML and deep learning benchmarks rely mainly on lag, rolling, and calendar-derived features. Future versions should test whether richer exogenous and spatial information improves performance. Planned extensions include:

- holiday and working-day features,
- cyclical calendar transformations,
- month-end and seasonal indicators,
- spatial embeddings from latitude and longitude,
- graph-based embeddings using regional proximity,
- distance-based features between markets,
- more systematic hyperparameter tuning for ML and deep learning models.

## Skills Demonstrated

- Time series forecasting
- Hierarchical forecasting
- Forecast reconciliation
- Walk-forward cross-validation
- Spatial feature engineering
- H3 and Geohash indexing
- Statistical modeling
- Machine learning benchmarks
- Deep learning benchmarks
- Error and bias analysis
- Python, R, StatsForecast, MLForecast, NeuralForecast, FoReco

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
