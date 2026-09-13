---
title: "NYC Traffic Data Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Reliability and congestion analysis of 4.2M NYC DOT traffic sensor readings across 5 boroughs, extended into a year-crossing XGBoost speed forecaster stress-tested against real baselines, boroughs, and horizons."
thumbnail: /assets/img/projects/nyc-traffic-congestion-sensor-map.jpg
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

### 1. Overview

I analyzed 4,233,169 rows of NYC DOT Traffic Speeds sensor data, from 2024-04-01 to 2024-08-01, to see how reliable the sensors actually are and how congestion really plays out across the five boroughs — then used that to test whether congestion could be predicted, and later extended it into a full year-crossing forecasting model.

<iframe src="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}" width="100%" height="600" style="border:0;"></iframe>

### 2. Problem

NYC's traffic is often called the most congested in the U.S. I wanted to see whether that congestion could be predicted well enough to find real solutions. But before I could get into any analysis or prediction, I realized I had to ask a more basic question first: could I even trust the sensor data I'd be working with?

### 3. Goals

My first goal was to check whether the areas assumed to be the most congested (Manhattan) actually were, and whether those same areas also had higher sensor error rates. Before testing machine learning or deep learning, I needed to know the data could be trusted, even in the most congested spots. In the end, the goal was to find out whether accurate prediction was something I could actually pull off in a realistic way.

### 4. Process

I measured reliability by boough as the percentage of records where status was not - 101. I removed segments with no valid readings, filled missing values using time-based interpolation owhile leaving status unchanged, and checked for values that stayed constant depsite a valid status. 

Peak-hour speeds were compared with each segment’s overnight free-flow speed. Lag correlations helped assess persistence as a forecasting baseline.
I later extended the analysis to 2024–Q1 2025, training XGBoost on 2024 and testing on Q1 2025. Evaluation covered 30- and 60-minute forecasts, persistence and “same time yesterday” baselines, and errors by borough and time of day.

### 5. Result

First, Queens had the highest reliability at 93.4%, while Manhattan had the lowest at 56.3%. 
| Borough       | Total rows | Error rate | Reliability |
| ------------- | ---------- | ---------- | ----------- |
| Manhattan     | 904,947    | 43.70%     | 56.3%       |
| Brooklyn      | 370,312    | 37.88%     | 62.1%       |
| Staten Island | 871,438    | 34.94%     | 65.1%       |
| Bronx         | 798,345    | 16.33%     | 83.7%       |
| Queens        | 1,288,127  | 6.64%      | 93.4%       |

<img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Sensor reliability by borough" style="max-width:100%;">

Error rates were lowest during rush hour and highest overnight, contraty to my expectation that errors would be more common during heavy traffic. 

Second, I compared peak-hour speeds with each borough's overnight free-flow speed. 

| Borough       | Free-flow, mph | PM peak, mph | Absolute drop |
| ------------- | -------------- | ------------ | ------------- |
| Manhattan     | 25.90          | 15.25        | **10.65**     |
| Staten Island | 55.78          | 41.54        | 14.24         |
| Queens        | 47.24          | 28.24        | 19.00         |
| Brooklyn      | 47.15          | 24.51        | 22.64         |
| Bronx         | 48.03          | 25.08        | 22.95         |

Manhattan’s speed dropped by 10.65 mph during the PM peak, compared with about 23 mph in Brooklyn and the Bronx. Its speeds were lower both overnight and during peak hours. The street network’s geographic features may have contributed to these lower speeds.

Third,congestion doesn't just disappear after one moment. In other words, it persists over time. The correlation between current speed and speed 30 minutes later was still 0.896. This shows temporal persistence, not congestion propagation between vehicles or segments. However, this phenomenon creates a real challenge for any model I'd want to build later. Since persistence already explains most of what happens 30 minutes out, a new model can't just be "pretty good" — it has to clearly beat that simple "it'll probably look like it does right now" guess to actually justify the added complexity.

Fourth, I trained XGBoost on 2024 and tested it on January-March 2025.

| Model                   | MAE   | RMSE  | R²    |
| ----------------------- | ----- | ----- | ----- |
| Naive (persistence)     | 5.994 | 9.881 | 0.658 |
| XGBoost (year-crossing) | 5.119 | 7.945 | 0.769 |

XGBoost outperformed Niave Model, reducing RMSE from 9.881 to 7.945 on the2025 test data. 
<img src="{{ '/assets/img/projects/nyc-traffic-year-over-year-performance.png' | relative_url }}" alt="Predicted vs actual, residual distribution, and monthly performance" style="max-width:100%;">

In the left plot, the model tended to predict higher speeds when acutal speeds were low. This means that it understimates how bad real congestion was. Although it performed better overall than basic model, it still needs improvement during heavy congestion. 

 
Fifth, I checked the 60-minute forecast horizon in more detail. The original lag analysis went straight from 30 minutes to one day, so I added the intervals in between. The correlation decreased gradually as the time gap increased.

| Lag (min) | 30 | 45 | 60 | 90 | 120 |
| --- | --- | --- | --- | --- | --- |
| Correlation | 0.77 | 0.72 | 0.68 | 0.60 | 0.53 |

Then, I compared 30-minute and 60-minute forecasts by borough.

<img src="{{ '/assets/img/projects/nyc-traffic-horizon-borough-comparison.png' | relative_url }}" alt="Lag correlation curve and 30min vs 60min RMSE by borough" style="max-width:100%;">

The 30 minutes had lower RMSE in all five boroughs, with reductions of 8–15%. It was more accurate, although the 60-minute model provided more advance notice.


Sixth, I tested whether persistence was really the toughest baseline out there, and pulled feature importance to see what the model was actually leaning on:

<img src="{{ '/assets/img/projects/nyc-traffic-report-extras.png' | relative_url }}" alt="Feature importance, baseline comparison, and peak vs off-peak error" style="max-width:100%;">



At the 30-minute horizon, the “same time yesterday” baseline had an RMSE of 12.2, compared with 8.6 for persistence. I expected yesterday’s traffic to be useful because of daily patterns, but using the current speed gave better predictions. Differences between weekdays and weekends may have contributed to this result. The feature importance also showed that the current speed and the 10-minute lag were more important than lag_1d.
Prediction errors were lower during peak hours (AM 6–9 / PM 15–19) than off-peak hours. This was unexpected, since I thought heavy traffic would be harder to predict. Recurring rush-hour patterns may have made prediction easier, though I did not test that explanation directly.

6. Reflection

I realized that heavy traffic can sometimes be easier to predict because it follows recurring patterns, while other factors may make off-peak speeds less predictable. I also learned that sensor reliability should be evaluated through data analysis rather than assumptions about how congested an area is.
