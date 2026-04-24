
# Predictive Paradox — Model Summary Report

## 1. Final Result
- **Model:** XGBoost Regressor
- **Test MAPE: 3.15%** (evaluated on full year 2023)
- **Test MAE:** 345.78 MW

## 2. Data Preparation & Cleaning

### Demand Data
- Removed 432 duplicate timestamps by keeping first occurrence
- Filtered data to 2015–2023 range, removing unreliable future entries
- Detected and replaced 76 extreme outlier spikes using a rolling 
  168-hour window Z-score method (threshold: 3 standard deviations)
- Outliers were replaced with rolling median values rather than dropped,
  preserving the continuity of the time series

### Weather Data  
- Raw file contained metadata rows at the top — skipped first 3 rows
  and reassigned correct column headers manually
- Filtered to match demand data date range (2015–2023)
- No missing values found after cleaning

### Economic Data
- Selected 5 most relevant macroeconomic indicators:
  GDP, Population, Industry Value, Electricity Access, Urban Population
- Transposed from wide format (years as columns) to long format
  (years as rows) for proper merging
- Joined to hourly data by matching calendar year — each hourly row
  inherits the annual economic value for its corresponding year

## 3. Feature Engineering

### Calendar Features
- Hour, day of week, month, quarter, day of year, is_weekend flag
- These capture daily and seasonal consumption patterns

### Cyclical Encoding
- Hour and month encoded as sin/cos pairs
- Reason: hour 23 is numerically close to hour 0 — raw integers
  mislead the model, sin/cos encoding fixes this

### Lag Features
- lag_1h, lag_2h, lag_3h, lag_6h, lag_12h: recent short-term behavior
- lag_24h, lag_48h: same hour on previous days (daily seasonality)
- lag_168h: same hour last week (weekly seasonality)
- These are the most important features — they give the model 
  memory of recent demand without violating tabular structure rules

### Rolling Window Features
- Rolling mean over 3h, 6h, 24h, 168h windows
- Rolling std over 24h, 168h windows
- Capture recent trend and volatility around the prediction point

## 4. Train/Test Strategy
- Strict chronological split — NO random shuffling
- Training: April 2015 to December 2022 (66,351 rows)
- Testing:  Full year 2023 (8,736 rows)
- All lag and rolling features computed using only past data
- Zero data leakage from future timestamps

## 5. Feature Importance Insights
Top drivers of electricity demand:
1. **lag_1h (0.59)** — The single strongest predictor. 
   Demand one hour ago is highly predictive of next hour demand
2. **lag_24h (0.16)** — Same hour yesterday captures daily rhythm
3. **hour_cos / hour_sin** — Time of day drives consumption patterns
4. **population** — Long term demand growth tied to population
5. **rolling_mean_3h** — Very recent trend captures momentum

Weather features (temperature, sunshine) contribute modestly,
suggesting Bangladesh grid demand is more calendar-driven than 
weather-driven at the hourly level.

## 6. Model Configuration
- Algorithm: XGBoost Regressor
- n_estimators: 1000, learning_rate: 0.05
- max_depth: 7, subsample: 0.8, colsample_bytree: 0.8
