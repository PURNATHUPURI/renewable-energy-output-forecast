# Renewable Solar/Wind Energy Output Forecast

![Python](https://img.shields.io/badge/python-3.9%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-regression-orange)
![Jupyter](https://img.shields.io/badge/notebook-Jupyter-F37626)
![SDG 7](https://img.shields.io/badge/UN%20SDG-7%20Clean%20Energy-yellow)
![Data](https://img.shields.io/badge/data-synthetic-lightgrey)

Machine learning project that forecasts the **hourly power output (MW)** of a 50 MWp solar plant and a 60 MW wind farm from weather, calendar signals and recent output, and compares four regressors against a persistence baseline.

> **Note:** the dataset is synthetic (physics-based simulation with noise and random outages). Results demonstrate the method and are not real-plant performance.

## Repository contents

| File | Description |
|---|---|
| `renewable_energy_forecast.ipynb` | End-to-end notebook: EDA, feature engineering, chronological split, 4 models, evaluation, cross-validation, feature importance (outputs included) |
| `renewable_energy_data.csv` | 17,544 hourly rows (2023-01-01 to 2024-12-31): weather inputs and solar/wind output |
| `Renewable_Energy_Project_Report.md` | Formal project report with problem statement, SDG alignment, data dictionary, leaderboards and recommendations |
| `README.md` | This file |

## Architecture

```
renewable_energy_data.csv
        |
        v
  EDA (patterns, KDE, power curves, correlation heatmap)
        |
        v
  Feature engineering
    - 7 weather variables
    - hour / day-of-year (sin, cos)
    - output 24 h ago, 24 h rolling mean (lagged 24 h)
        |
        v
  Chronological split
    train 2023-01 -> 2024-06   |   test 2024-07 -> 2024-12
        |
        v
  Models: Persistence | Linear Regression | Decision Tree | Random Forest | Gradient Boosting
        |
        v
  Evaluation: MAE, RMSE, R2, nRMSE + time-series cross-validation + feature importance
```

## Installation

```bash
git clone <your-repository-url>
cd <your-repository-folder>

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook renewable_energy_forecast.ipynb
```

Run all cells from top to bottom. The full notebook (including 4-fold cross-validation) takes a couple of minutes on a laptop.

## Dataset

| Column | Unit | Description |
|---|---|---|
| `timestamp` | hourly | Local time |
| `temperature_c` | deg C | Air temperature |
| `humidity_pct` | % | Relative humidity |
| `pressure_hpa` | hPa | Air pressure |
| `cloud_cover_pct` | % | Cloud cover |
| `ghi_wm2` | W/m2 | Global horizontal irradiance |
| `wind_speed_ms` | m/s | Wind speed at 100 m |
| `wind_direction_deg` | degrees | Wind direction |
| `solar_output_mw` | MW | **Target** - solar output |
| `wind_output_mw` | MW | **Target** - wind output |
| `total_output_mw` | MW | Solar + wind |

## Benchmark results (held-out test: Jul-Dec 2024, 4,416 hours)

### Solar output

| Model | MAE (MW) | RMSE (MW) | R2 | nRMSE |
|---|---|---|---|---|
| Gradient Boosting | 0.277 | **1.341** | 0.985 | 2.68% |
| Linear Regression | 0.473 | 1.348 | 0.985 | 2.70% |
| Random Forest | 0.272 | 1.352 | 0.985 | 2.70% |
| Decision Tree | 0.316 | 1.434 | 0.983 | 2.87% |
| Persistence (24 h) | 1.468 | 3.530 | 0.898 | 7.06% |

### Wind output

| Model | MAE (MW) | RMSE (MW) | R2 | nRMSE |
|---|---|---|---|---|
| Random Forest | 1.009 | **3.523** | 0.954 | 5.87% |
| Gradient Boosting | 1.043 | 3.667 | 0.950 | 6.11% |
| Decision Tree | 1.162 | 3.954 | 0.942 | 6.59% |
| Linear Regression | 3.898 | 5.766 | 0.876 | 9.61% |
| Persistence (24 h) | 13.098 | 18.022 | -0.214 | 30.04% |

### Takeaways

- Every model beats the persistence baseline on both targets.
- **Wind** benefits strongly from non-linear models (Random Forest RMSE 3.52 MW vs 5.77 MW for Linear Regression) because of the turbine power curve.
- **Solar** is almost linear in irradiance, so all four models are within about 0.1 MW RMSE; the tree models mainly improve MAE.
- Irradiance (solar) and wind speed (wind) account for roughly 98-99% of Gradient Boosting feature importance.

## Limitations

- Synthetic data generated from the same physical relationships the models learn, so scores are optimistic.
- Weather inputs are observed values, not real forecasts; real forecast error would increase output error.
- Single location, two years of data.

## Next steps

- Train and test on real SCADA data with archived weather-forecast inputs
- Add probabilistic (quantile) forecasts
- Try LightGBM and sequence models (LSTM)

## License

Add a license of your choice (for example MIT) before publishing.
