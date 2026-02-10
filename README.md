# TimeFlight — Regional Flight Delay Forecasting

## TL;DR
TimeFlight is a regional flight-delay forecasting system built on 21 years of U.S. aviation data.  
Using historical delay minutes from the Bureau of Transportation Statistics, the project shows that:

- Flight delays are **not uniform across the U.S.** and must be modeled region-by-region  
- **Seasonality is the dominant signal** driving delay risk nationwide  
- The **South East and West** experience the highest and most volatile delay pressure  
- Classical time-series models, when applied correctly, can **reliably flag high-risk months in advance**  
- **Seasonal Auto-ARIMA** consistently produces the most stable and operationally useful forecasts  

The output is not a perfect point forecast, but a **directional risk signal** that supports staffing, scheduling, and contingency planning

---

## Problem Context
Flight delays impose tens of billions of dollars in annual losses across airlines, airports, travelers, and the wider economy.  
Despite this, many operational decisions are still based on heuristics like “summer is bad” rather than quantified, forward-looking risk.

This project asks a focused question:  
*Can we use historical delay behavior alone to generate early, region-specific signals of future delay pressure that are useful for real planning decisions?*

---

## Key Results
The analysis shows clear and persistent regional structure in delay behavior.

- The **South East and West** consistently exhibit higher average delays and larger volatility  
- The **North East and South West** are comparatively stable and serve as useful baselines  
- All regions show strong **annual seasonality**, with predictable summer peaks  
- Structural shocks, especially COVID-19, materially shift baseline behavior and must be handled explicitly  

Across regions and historical cutoffs, **Seasonal Auto-ARIMA** best balances accuracy, stability, and interpretability, particularly in high-volatility regions.

---

## Dataset
TimeFlight uses the **Airline On-Time Performance / Cause of Delay** dataset from the  
U.S. Bureau of Transportation Statistics (BTS).

- Time period: **2004–2024**  
- Granularity: **Monthly, airport-level observations**  
- Size: ~400,000 rows across all U.S. airports  

The raw data is aggregated into **five U.S. regions** (North East, Mid West, South East, South West, West) to improve signal-to-noise and support regional planning.

**Dataset source:**  
https://www.transtats.bts.gov/

---

## Approach (High Level)
The project follows a deliberately lightweight and transparent pipeline:

1. Ingest and clean monthly delay data from BTS  
2. Map airports into five geographic regions  
3. Aggregate airport-level delays into region-level time series  
4. Perform exploratory analysis to understand trend, volatility, and seasonality  
5. Forecast each region independently using classical univariate models:
   - Double ETS  
   - Triple ETS  
   - ARIMA(1,0,1)  
   - Auto-ARIMA (seasonal and non-seasonal)  
6. Evaluate models across multiple historical cutoffs to test robustness  

The focus is on **stable, interpretable signals**, not overfitting.

---

## Repository Contents
- `TimeFlight Final Code`  
  End-to-end analysis: data preparation, EDA, modeling, and evaluation.
- `TimeFlight Final Report`  
  Detailed write-up of assumptions, model comparisons, and findings.
- `TimeFlight Presentation`  
  Executive-level framing focused on operational and policy impact.

Each component reinforces a different layer of understanding: code, analysis, and decision context.

---

## How to Use
This repository is designed for exploration and review.

1. Start with the presentation to understand the problem and outcomes  
2. Read the report for deeper reasoning and model comparison  
3. Review the code to see how the pipeline is implemented end-to-end  
4. Extend the analysis by adding exogenous variables or airport-level modeling  

---

## Why This Matters
TimeFlight demonstrates that **simple, well-designed models** can deliver meaningful operational value when paired with strong data engineering and evaluation discipline.

Rather than chasing complexity, the project shows how classical forecasting methods can still play a central role in real-world decision support systems.

---

## Next Steps
Planned extensions include:
- Incorporating weather, traffic volume, and scheduling data  
- Moving from regional to airport-level forecasting  
- Supporting additional delay metrics beyond total delay minutes  
- Packaging forecasts into dashboards or APIs for operational use  
