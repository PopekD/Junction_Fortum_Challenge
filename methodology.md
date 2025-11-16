# Methodology and Implementation Report: High-Dimensional Load Forecasting

## 1. Data Sources and Preparation

This section outlines the data used, including external sources, resolution choices, and initial processing steps necessary to prepare the time series for modeling.

### 1.1 Data Characteristics and Scope

The project utilized approximately four years ($\sim 35,000$ hourly data points) of historical energy consumption data across 112 distinct geographical regions in Finland (the primary load data).

External Data: Weather information (minimum daily temperature, maximum daily temperature) was sourced from the Finnish Meteorological Institute across 30 distinct observation stations. The decision to incorporate temperature was based on visual data inspection and the intuitive knowledge of Finland's harsh, seasonal climate, which heavily influences heating/cooling load.

### 1.2 Data Augmentation and Imputation

Weather Resolution: The weather data was confined to daily granularity (min/max temperature) to simplify feature engineering across the 112 regions. These daily values were then dispersed to all hourly records within the corresponding region.

Missing Data Imputation: Any missing data points in the temperature time series were imputed using a seasonal substitution method, specifically by using the observed minimum and maximum temperatures from the corresponding day in the next chronological year.

### 1.3 Monthly Data Aggregation

To address the challenge of predicting the annual total load, a separate dataset was created by aggregating the hourly data month-wise. This process collapsed the hourly data into 48 distinct month-instance rows (one row per Year-Month combination across the four-year span).

## 2. Feature Engineering and Autoregressive Setup

The core modeling strategy relied on transforming the time series problem into a Multivariate, Multi-Output Regression task using targeted autoregressive features.

### 2.1 Feature Vector Composition

The input feature vector for the model was constructed to provide both weather context and short-term autoregressive information:

Local and Short-Term Lags: For each of the 112 regions, the input included the consumption at $t-1$ (one hour ago), $t-2$ (two hours ago), and $t-24$ (same hour yesterday). This explicitly bakes in local trends and short-term daily seasonality.

Exogenous Variables: Regional minimum temperature, maximum temperature, and the prevailing energy price (exact where known, imputed via historical averages where not).

Vector Construction: All regional and exogenous features were concatenated into a single, high-dimensional feature vector. This design allowed the model to predict all 112 next hour's regional consumption values in parallel (multivariate output).

### 2.2 Monthly Feature Set

For the monthly forecasting challenge, the high-frequency temporal lags were removed. The historical context was instead encoded via a 12-month lag, along with the aggregated mean weather variables for the corresponding month.

## 3. Modeling and Validation

### 3.1 Model Selection

The chosen model for both hourly and monthly challenges was the Decision Tree Regressor (specifically, the RandomForestRegressor was considered, but the simpler single Decision Tree was selected).

Justification: The Decision Tree was selected for its speed of iteration, its theoretical capacity for arbitrarily partitioning the decision space (useful in high dimensions), and its ability to act as a readily interpretable, multi-output regressor in the Scikit-learn framework.

### 3.2 Training and Validation Strategy

Hourly Data Validation: A standard time-based validation split was employed, using the last 30% of the dataset for testing and the remainder for training. This non-random chronological split was chosen to prevent data leakage due to the use of temporal lags.

Inference: Prediction was performed autoregressively, feeding the model's own predictions back into the feature set for subsequent time steps.

Performance: The model reported an MSE of $0.05$ (unnormalized) on the validation set.

Monthly Data Validation: Due to the limited size of the monthly dataset, Leave-One-Out Cross-Validation (LOOCV) was utilized to achieve the most rigorous estimation of model generalization error.

Performance: LOOCV yielded a Mean Absolute Error (MAE) of $91$ (unnormalized).

## 4. Results and Conclusions

The modeling approach yielded visually strong results, accurately capturing the historical and structural trends in the hourly load data.

Hourly Performance: The visual inspection (as demonstrated in figures) confirmed that the Decision Tree successfully predicts the correct daily profile, indicating the feature engineering effectively translated temporal and weather context into predictable feature splits. The prediction accuracy, while based on a complex non-linear model, demonstrated high fidelity to the underlying load curve.

Conclusion: The success of the model highlights the significance of the autoregressive feature design and the critical role of external weather data in load forecasting, even when using a structurally simple, multi-output regression model like the Decision Tree.


Daily trend:
![](/figs/true.png)


Predicted daily trend using only autoregressive data:
![](/figs/pred.png)