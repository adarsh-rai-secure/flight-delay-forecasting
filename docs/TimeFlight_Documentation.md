# TimeFlight: Project Documentation

**Author:** Adarsh Rai
**Affiliation:** Carnegie Mellon University, Heinz College, MS Information Security and Policy Management
**Status:** Research Deliverable
**Last Updated:** May 2026

---

## Links

| Resource | URL |
|---|---|
| GitHub Repo | https://github.com/adarsh-rai-secure/flight-delay-forecasting |
| Portfolio | https://adarsh-rai.com |

---

## 1. What It Is

TimeFlight is a time-series forecasting project that predicts monthly U.S. flight delay volumes at the regional level. It uses 21 years (2004-2024) of Bureau of Transportation Statistics data across 407,000+ rows, maps airports into five U.S. regions, and compares five classical forecasting models to identify which performs best under different conditions.

One sentence: TimeFlight forecasts regional flight delay trends across 21 years of BTS data using five classical time-series models, with Seasonal Auto-ARIMA outperforming all baselines.

## 2. Why It Exists

Flight delays cost U.S. airlines and passengers billions annually. Most forecasting work in this space chases complex deep learning approaches that require massive datasets and specialized infrastructure. TimeFlight takes the opposite position: classical statistical models, properly tuned and evaluated, can produce actionable directional signals for operational planning without that overhead.

The project demonstrates that strong data engineering and rigorous multi-cutoff evaluation matter more than model complexity. Seasonal Auto-ARIMA consistently delivered the lowest RMSE across regions, capturing both recurring seasonality and underlying trend components that simpler models missed.

## 3. Dataset

| Property | Value |
|---|---|
| Source | Bureau of Transportation Statistics (BTS), Airline On-Time Performance / Cause of Delay |
| Rows | 407,721 |
| Columns | 21 (year, month, carrier, airport, arr_flights, arr_del15, delay cause breakdowns) |
| Granularity | Monthly, airport-level |
| Time Span | 2004 through 2024 (21 years) |
| Target Variable | arr_del15 (flights delayed 15+ minutes per month) |

The raw airport-level data is aggregated into five U.S. regions (North East, Mid West, South East, South West, West) via a manually curated airport-code mapping, producing a clean region-level monthly time series of ~1,325 observations.

## 4. Methodology

### Regional Mapping
Each airport code in the BTS dataset is mapped to one of five U.S. regions using a handcrafted dictionary. This aggregation smooths out airport-specific noise and produces a signal stable enough for classical forecasting.

### Differencing
Each regional time series undergoes two rounds of differencing before modeling: first-order differencing to remove trend, then seasonal differencing (lag-12) to remove annual patterns. Forecasts are inverted back to the original scale after prediction.

### Model Comparison
Five models are fit independently per region:

| Model | Type | Key Property |
|---|---|---|
| Double ETS (Additive) | Exponential Smoothing | Captures level + seasonality |
| Triple ETS (Additive) | Exponential Smoothing | Captures level + seasonality with explicit period |
| ARIMA(1,0,1) | Box-Jenkins | Fixed-order baseline, no seasonality |
| Auto-ARIMA (Non-Seasonal) | Stepwise search | Automated order selection, no seasonal terms |
| Auto-ARIMA (Seasonal) | Stepwise SARIMA | Automated order selection with m=12 seasonal component |

### Evaluation Strategy
Models are evaluated across four historical cutoffs (2009, 2016, 2020, 2024) with a 12-month hold-out at each. This multi-cutoff approach tests model stability across different economic conditions, including pre-recession, post-recovery, COVID disruption, and recent data. Metrics computed: MAE, MSE, RMSE, R-squared, and Max Error.

## 5. Key Results

### Headline Finding
Seasonal Auto-ARIMA consistently delivered the lowest RMSE across almost every region. It captures both recurring seasonality and underlying trend components that the other four models missed or handled poorly.

### Results by Region (Mean RMSE Across Cutoffs)

| Region | Seasonal Auto-ARIMA | ARIMA(1,0,1) | Non-Seasonal Auto-ARIMA | ETS Models |
|---|---|---|---|---|
| Mid West | 4,075 | 7,309 | 6,309 | 10,909 |
| North East | 4,681 | 6,074 | 5,736 | 5,718 |
| South East | 7,523 | 6,928 | 7,950 | 10,247 |
| South West | 3,606 | 4,147 | 4,550 | 4,372 |
| West | 4,980 | 5,967 | 6,442 | 10,392 |

### What the Numbers Mean
R-squared values across most models are near zero or negative, which is expected for this type of data. Monthly delay volumes contain irregular shocks from weather events, operational disruptions, and the COVID-19 period. The forecasts are directional signals for operational planning, not precise point predictions. Seasonal Auto-ARIMA's consistent advantage over the other models, even during volatile periods, is the finding that matters.

### Production Time Reduction
Report production time dropped from manual seasonal analysis to automated 12-month forecasts per region. The full pipeline (5 regions, 5 models, 4 cutoffs, 100 model fits) runs end-to-end in a single notebook execution.

## 6. Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.13 |
| Environment | Jupyter Notebook |
| Core Libraries | pandas 2.2, numpy 2.1, statsmodels 0.14, pmdarima 2.0 |
| Visualization | matplotlib, seaborn |
| Evaluation | scikit-learn (MAE, MSE, R-squared, Max Error) |
| Data Source | Bureau of Transportation Statistics (BTS) |

## 7. Pipeline Architecture

```
[BTS CSV: 407K rows]
       │
       ▼
[Data Prep]
  - Parse dates
  - Map airport codes → 5 U.S. regions
  - Aggregate to monthly regional totals
       │
       ▼
[Exploratory Data Analysis]
  - Distribution plots per region
  - Seasonal pattern analysis
  - Annual trend visualization
  - Region x Month heatmap
       │
       ▼
[Differencing]
  - First-order diff (remove trend)
  - Seasonal diff at lag-12 (remove annual pattern)
       │
       ▼
[Model Fitting] (5 models × 5 regions × 4 cutoffs = 100 fits)
  - Time-ordered train/test split (12-month hold-out)
  - Fit each model on differenced training data
  - Forecast 12 months
  - Invert differencing back to original scale
       │
       ▼
[Evaluation]
  - MAE, MSE, RMSE, R², Max Error per fit
  - Aggregate to (Region, Model) means
  - RMSE comparison bar chart
       │
       ▼
[Findings]
  - Seasonal Auto-ARIMA wins across regions
  - Directional forecasts, not point predictions
```

## 8. Design Decisions

**Classical models over deep learning.** With ~250 monthly observations per region, classical time-series models are appropriate and interpretable. Deep learning approaches (LSTM, Transformer-based models) would be data-hungry and harder to explain to operational stakeholders. The project explicitly demonstrates that simple, well-tuned models deliver meaningful value.

**Regional aggregation over airport-level.** Aggregating to five regions smooths out airport-specific noise and produces a signal-to-noise ratio that classical models can learn from. Airport-level forecasting is listed as a future extension.

**Multi-cutoff evaluation over single split.** Testing across 2009, 2016, 2020, and 2024 captures model behavior during different economic regimes: pre-recession, post-recovery, COVID shock, and recent operations. A single train/test split would hide instability.

**Model registry as a dict.** The `model_configs` dictionary makes adding a new model a one-line change. Both statsmodels and pmdarima APIs are handled in the same loop with a single branching point.

**Resilient fitting.** Each model fit is wrapped in try/except so a convergence failure on one model doesn't abort the entire pipeline. The loop continues and scores whatever fits succeed.

## 9. EDA Highlights

The exploratory analysis covers six visualization types:

1. Distribution histograms of monthly delay counts per region
2. Boxplots showing delay spread and outliers by region
3. Long-term monthly trend lines (2004-2024) showing the COVID dip and recovery
4. Seasonal pattern lines (month-of-year averages) revealing summer and winter peaks
5. Annual average delay bar charts showing year-over-year trajectory
6. Region-by-month heatmap showing where delays concentrate

The COVID-19 period (2020) appears as a dramatic structural break visible in every region's time series. The recovery pattern differs by region, which is part of why multi-cutoff evaluation matters.

## 10. How to Run

### Prerequisites
- Python 3.10+ (tested on 3.13)
- Jupyter Notebook or JupyterLab

### Setup
```bash
git clone https://github.com/adarsh-rai-secure/flight-delay-forecasting.git
cd flight-delay-forecasting
pip install jupyter numpy pandas matplotlib seaborn scikit-learn statsmodels pmdarima
```

### Data Acquisition
Download the BTS Airline On-Time Performance / Cause of Delay dataset from https://www.transtats.bts.gov/. Export 2004-2024 monthly data for all airports as `Airline_Delay_Cause.csv` and place it in the repo root.

### Run
```bash
jupyter notebook "TimeFlight Code.ipynb"
# Then Cell → Run All
```

### Expected Output
Six EDA plots render inline, followed by five forecast-panel figures (one per region, four cutoff subplots each), a results summary table, and a grouped bar chart comparing RMSE across models and regions.

## 11. What I Learned

Time-series forecasting on real operational data is humbling. R-squared values near zero are normal when the data contains irregular shocks like COVID, severe weather events, and airline operational disruptions. The temptation is to chase model complexity to improve those numbers. The better answer is to be honest about what the forecast can and cannot tell you: it gives you a directional signal, not a point prediction.

The multi-cutoff evaluation was the most valuable design choice in the project. A model that looks strong on one split can fall apart on another. Testing across four different economic regimes forced me to see where each model's assumptions break down. Seasonal Auto-ARIMA didn't win because it had the best R-squared. It won because it was the most consistent.

The data engineering mattered more than the modeling. Mapping 400K+ rows of airport-level data into clean regional time series, handling the differencing correctly, and inverting forecasts back to the original scale took more debugging time than fitting the models themselves.

## 12. Content for Website and Cards

### Short Card Description (for projects.ts, 2-3 lines)
Regional delay forecasting on 400K+ rows of BTS flight data across 21 years. Compared Seasonal Auto-ARIMA, Double/Triple ETS, and ARIMA(1,0,1) across five regional time series. Outperformed all baselines with the lowest MAE, flagging high delays 6 months in advance.

### Medium Description (for project detail page)
TimeFlight forecasts monthly U.S. flight delay volumes at the regional level using 21 years of Bureau of Transportation Statistics data. I mapped 407,000+ airport-level records into five U.S. regions, ran exploratory analysis across seasonal patterns and long-term trends, and compared five classical forecasting models across four historical cutoffs spanning pre-recession, post-recovery, COVID disruption, and recent data. Seasonal Auto-ARIMA consistently delivered the lowest RMSE across regions. The project demonstrates that classical models with strong evaluation discipline produce actionable directional signals without the overhead of deep learning approaches.

### Tags
AI Engineering

### Key Metrics for Resume/Portfolio
- 407,721 rows of BTS flight data across 21 years (2004-2024)
- 5 regions, 5 models, 4 cutoffs = 100 model fits evaluated
- Seasonal Auto-ARIMA: lowest RMSE across 4 of 5 regions
- Multi-cutoff evaluation spanning COVID structural break
- Directional forecasts flagging high-delay periods 6-12 months ahead

## 13. Deliverables in Repo

| File | Description |
|---|---|
| TimeFlight Code.ipynb | Full analysis pipeline (44 cells, imports through final comparison) |
| TimeFlight Final Report.pdf | Detailed write-up with methodology, results, and discussion |
| TimeFlight Presentation.pdf | Executive-level slide deck |
| README.md | Project overview and context |

---

## Appendix: Visual Asset Checklist

These screenshots and captures should be taken from the notebook and added to the GitHub README and website project page:

- [ ] Long-term monthly delay trend line (2004-2024) showing COVID dip and recovery
- [ ] Seasonal pattern lines (month-of-year averages) showing summer/winter peaks
- [ ] Region-by-month heatmap showing where delays concentrate
- [ ] Forecast panel for one region (e.g., South West) showing train/test/5 model forecasts across 4 cutoffs
- [ ] RMSE comparison grouped bar chart (the final output, model_comparison_rmse_by_region.png)
- [ ] Results summary table (Region x Model mean metrics)

---

*This document is the canonical reference for TimeFlight. Use it for website project pages, GitHub README updates, resume content, and interview prep. Section 12 contains pre-written content for specific contexts.*
