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

### Explore the map

Every segment on the network, colored by borough, with line thickness showing PM-peak congestion and two separate hotspot markers — orange for the most congested segments, black for sensor error hotspots. Hover any line or marker for exact values.

<iframe src="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}" width="100%" height="600" style="border:0;"></iframe>

### 2. Problem

NYC's traffic is often called the most congested in the U.S. I wanted to see whether that congestion could be predicted well enough to find real solutions. But before I could get into any analysis or prediction, I realized I had to ask a more basic question first: could I even trust the sensor data I'd be working with?

### 3. Goals

My first goal was to check whether the areas assumed to be the most congested (Manhattan) actually were, and whether those same areas also had higher sensor error rates. Before testing machine learning or deep learning, I needed to know the data could be trusted, even in the most congested spots. In the end, the goal was to find out whether accurate prediction was something I could actually pull off in a realistic way.

### 4. Process

I defined reliability as (total rows − rows with `status == -101`) / total rows and checked it by borough before trusting a single network-wide number. Then I dropped the unused columns, cut the segments that never reported one valid reading in 4 months, and filled the rest with time-based interpolation — but I kept `status` itself untouched by that, since interpolating it too would've made reliability look better than it actually is. I also added a stuck-run check for segments that report a "valid" status while the value itself never moves, since status codes alone weren't catching that.

When I looked at peak hours by borough, Manhattan came out as the slowest overall — so I checked it against each segment's own overnight free-flow speed instead of just comparing raw speeds across boroughs. Then, before touching any forecasting model, I built the lag/rolling features it would need and checked lag correlation first, so I'd know upfront how hard a baseline any model would actually have to beat.

That's as far as the original 4-month analysis went. I later extended it: pulled the full 2024–2025Q1 dataset, trained an XGBoost forecaster on all of 2024 and tested it on 2025 (a year it never saw), then went back and checked three things I'd been assuming instead of actually testing — the 60-minute prediction horizon, whether persistence was really the hardest baseline available, and whether the model was uniformly good or just good on average.

### 5. Result

**First**, Queens came out as the most reliable borough overall (93.4%), way above Manhattan's 56.3%:

| Borough       | Total rows | Error rate | Reliability |
| ------------- | ---------- | ---------- | ----------- |
| Manhattan     | 904,947    | 43.70%     | 56.3%       |
| Brooklyn      | 370,312    | 37.88%     | 62.1%       |
| Staten Island | 871,438    | 34.94%     | 65.1%       |
| Bronx         | 798,345    | 16.33%     | 83.7%       |
| Queens        | 1,288,127  | 6.64%      | 93.4%       |

<img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Sensor reliability by borough" style="max-width:100%;">

My guess is this comes down to geography — Queens is the largest borough by area and its network is a lot more spread out, highways criss-crossing instead of one dense grid, so a single dead segment doesn't sink the borough's average the way it can in Manhattan. That's just my interpretation though, not something I directly tested in the data. What I did test was the obvious follow-up hypothesis — that heavy traffic on famously congested corridors wears sensors down — and the data said the opposite: error rate was lowest during rush hour and highest overnight. So congestion doesn't explain sensor failure; whatever's driving it is something else, tied to specific hours a given segment happens to report at all.

**Second**, I think Manhattan's congestion is really a geography problem, not a rush-hour problem. Comparing peak-hour speed against each segment's own overnight free-flow speed:

| Borough       | Free-flow, mph | PM peak, mph | Absolute drop |
| ------------- | -------------- | ------------ | ------------- |
| Manhattan     | 25.90          | 15.25        | **10.65**     |
| Staten Island | 55.78          | 41.54        | 14.24         |
| Queens        | 47.24          | 28.24        | 19.00         |
| Brooklyn      | 47.15          | 24.51        | 22.64         |
| Bronx         | 48.03          | 25.08        | 22.95         |

Manhattan only drops about 10.65 mph at peak, while Brooklyn and the Bronx drop 19–23 mph. So Manhattan isn't actually getting worse at rush hour — it's just slow all day, all the time, because it's built on a dense street grid instead of highways, which means constant signal stops adding delay no matter what time it is. The other boroughs are highway-fast overnight and collapse hard specifically at peak — they're the ones actually carrying a rush-hour congestion problem, not Manhattan.

**Third**, congestion doesn't just disappear after one moment — it carries over. The correlation between current speed and speed 30 minutes later was still 0.896, so whatever's happening right now is a pretty strong predictor of what happens in the next half hour. That's actually a problem for any model I'd want to build later, since a model would need to beat that simple "it'll probably look like it does right now" guess to actually be worth using.

**Fourth**, when I actually built that forecaster (XGBoost, trained on all of 2024, tested on Jan–Mar 2025 — a year it never saw), it did beat persistence, and by a real margin:

| Model                  | MAE   | RMSE  | R²    |
| ----------------------- | ----- | ----- | ----- |
| Naive (persistence)     | 5.994 | 9.881 | 0.658 |
| XGBoost (year-crossing) | 5.119 | 7.945 | 0.769 |

That's unexplained variance dropping from 34.2% to 23.1%, and it held up on a full year gap, not just a lucky split of the same few months. But averages hide where a model fails, and this one has a real weak spot:

<img src="{{ '/assets/img/projects/nyc-traffic-year-over-year-performance.png' | relative_url }}" alt="Predicted vs actual, residual distribution, and monthly performance" style="max-width:100%;">

Looking at predicted vs. actual (left panel), at low actual speeds — real congestion — the predictions cluster well above the diagonal. The model systematically underestimates how bad congestion actually gets, which shows up again as a long positive tail in the residuals. So it's good at typical traffic and not yet good at the moments a congestion alert would actually need to catch.

**Fifth**, I went back and questioned the 60-minute horizon I'd been using, since my own lag-correlation table jumped straight from 30 minutes to 1 day with nothing measured in between. I filled that gap and found the correlation just keeps decaying smoothly through it — no cliff at 30 minutes, just a signal that keeps eroding:

| Lag (min) | 30 | 45 | 60 | 90 | 120 |
| --- | --- | --- | --- | --- | --- |
| Correlation | 0.77 | 0.72 | 0.68 | 0.60 | 0.53 |

That made me actually train a 30-minute model side by side with the 60-minute one and break both down by borough:

<img src="{{ '/assets/img/projects/nyc-traffic-horizon-borough-comparison.png' | relative_url }}" alt="Lag correlation curve and 30min vs 60min RMSE by borough" style="max-width:100%;">

30 minutes won in all 5 boroughs, no exceptions — RMSE dropped 8–15% depending on the borough. So if the actual goal is a borough-level congestion signal, 30 minutes is the defensible choice, not 60 — I'd been defaulting to 60 without ever checking whether it was the right call.

**Sixth**, I also checked whether persistence was really the toughest baseline available, and pulled feature importance to see what the model was actually leaning on:

<img src="{{ '/assets/img/projects/nyc-traffic-report-extras.png' | relative_url }}" alt="Feature importance, baseline comparison, and peak vs off-peak error" style="max-width:100%;">

Two things here I didn't expect. I tried a "same time yesterday" baseline, reasoning that the 1-day lag correlation (0.778) actually beats lag times past ~40 minutes, so it seemed like it should be a stronger competitor than plain persistence. It wasn't — it came out clearly worse (RMSE 12.2 vs. 8.6 at the 30-minute horizon), because "yesterday" isn't always the same day of the week (a Monday's yesterday is a Sunday), and that mismatch costs more than the extra correlation buys. The model's own feature importance agrees — `lag_1d` barely registers next to the current reading and the 10-minute lag. And peak hours (AM 6–9 / PM 15–19) turned out easier to predict than off-peak, not harder, which I also wasn't expecting going in — rush-hour congestion is a strong, recurring pattern the model has clearly learned, while off-peak variability is sparser and more incident-driven, and harder to pin down.

So where this leaves things: persistence is still the real competitor, and the model beats it convincingly across a full year gap, in every borough, at the horizon that actually makes sense. It's reliable for typical traffic and for the recurring rush-hour pattern — it's just not a congestion-severity detector yet, which is the next thing I'd actually want to build.
