---
title: "NYC Traffic Data Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Reliability and congestion analysis of 4.2M NYC DOT traffic sensor readings across 5 boroughs, uncovering why sensors fail and how congestion actually differs by borough."
thumbnail: /assets/img/projects/nyc-traffic-congestion-sensor-map.jpg
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

Analyzed 4,233,169 rows of NYC DOT Traffic Speeds (NBE) sensor data — 5 boroughs, 2024-04-01 to 2024-08-01 — to answer two questions: how reliable are the sensors, and how does congestion actually vary across the city?

### Explore the map

Every segment on the network, colored by borough, with line thickness showing PM-peak congestion and two separate hotspot markers — orange for the most congested segments, black for sensor error hotspots. Hover any line or marker for exact values.

<iframe src="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}" width="100%" height="600" style="border:0;"></iframe>

### 1. Sensor reliability

Reliability = (total rows − rows with `status == -101`) / total rows. Network-wide: **75.05%** (1,056,062 error rows out of 4,233,169).

| Borough | Total rows | Error rate | Reliability |
|---|---|---|---|
| Manhattan | 904,947 | 43.70% | 56.3% |
| Brooklyn | 370,312 | 37.88% | 62.1% |
| Staten Island | 871,438 | 34.94% | 65.1% |
| Bronx | 798,345 | 16.33% | 83.7% |
| Queens | 1,288,127 | 6.64% | 93.4% |

Queens is the most reliable, Manhattan the least — a 6.6x gap. In Manhattan, Brooklyn, Staten Island, and Bronx, most of the worst segments sit at 100% error (fully dead sensors, e.g. the Verrazzano-Narrows Bridge, Major Deegan Expwy, West Shore Expwy, Staten Island Expwy, West Side Highway, Lincoln Tunnel), whereas Queens's worst segments are only partially unstable (19–33% error, none fully dead) — which is why Queens comes out on top overall.

<img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Sensor reliability by borough" style="max-width:100%;">

**Does congestion cause sensor failure?** I tested the obvious hypothesis — that heavy traffic on famously congested corridors (GWB approach, Lincoln Tunnel, West Side Highway) wears sensors down — on the two partially-dead Manhattan segments that still had some valid readings. The result was the opposite of what the hypothesis predicts: error rate was *lowest* during rush hour (72–92%) and *highest* overnight (~100%). **Rejected** — congestion doesn't explain sensor failure. What actually varies is which hours of the day a given segment happens to report valid data at all, a window that has to be checked per segment rather than assumed.

### 2. Data processing

- Dropped duplicate/unused columns (`link_id`, `transcom_id`, `encoded_poly_line`, `encoded_poly_line_lvls`).
- Removed **16 fully-dead segments** (487,811 rows) that never reported a single valid reading in 4 months — nothing to anchor an interpolation to.
- For the remaining segments, `status == -101` rows were set to NaN and filled with time-based interpolation, falling back to `ffill`/`bfill` at the edges. Missing `speed`/`travel_time` went from ~396K rows to 0.
- Final row count: 4,233,169 → **3,745,358**.

Interpolation only fills `speed`/`travel_time`, never `status` — so the reliability numbers above are always computed from the raw data, not the cleaned output, otherwise the denominator would shrink and reliability would look artificially better.

### 3. Time series analysis

Peak hour by borough (weekday, lowest average speed within the AM 6–9 / PM 15–19 windows):

| Borough | AM peak | PM peak |
|---|---|---|
| Bronx | 9am (31.8 mph) | 4pm (25.1 mph) |
| Brooklyn | 8am (29.6 mph) | 4pm (24.5 mph) |
| Manhattan | 9am (18.7 mph) | 4pm (15.3 mph) |
| Queens | 8am (32.8 mph) | 4pm (28.2 mph) |
| Staten Island | 8am (46.4 mph) | 5pm (40.8 mph) |

Four boroughs share the exact same PM peak hour (4pm); only Staten Island differs (5pm), and runs far faster overall. Staten Island has no subway — commuting runs through the Verrazzano Bridge and the ferry — so its congestion timing plausibly follows a structurally different pattern from the other four subway-served boroughs.

**Is Manhattan really the most congested?** Using each segment's 2–4am average speed as a free-flow baseline:

| Borough | Free-flow, mph | PM peak, mph | Absolute drop |
|---|---|---|---|
| Manhattan | 25.90 | 15.25 | **10.65** |
| Staten Island | 55.78 | 41.54 | 14.24 |
| Queens | 47.24 | 28.24 | 19.00 |
| Brooklyn | 47.15 | 24.51 | 22.64 |
| Bronx | 48.03 | 25.08 | 22.95 |

Manhattan is slowest in absolute terms at every hour of the day, but has the *smallest* drop from its own baseline — because that baseline is already low (a low-speed-limit street grid, not a highway). Manhattan isn't "worst at peak," it's **uniformly slow all day** — a structural property of the road network rather than a rush-hour phenomenon. Brooklyn, Bronx, and Queens, by contrast, are highway-fast overnight (~47–48 mph) and collapse hardest at peak (19–23 mph drop) — they carry the network's actual peak-hour congestion problem.

### 4. ML/DL feature engineering

Built lag/rolling features for downstream forecasting (`build_ml_feature_df()`): each segment is first reindexed onto a regular 5-minute time grid (gaps time-interpolated up to a limit, longer gaps left as `NaN`) so that a shift actually corresponds to a fixed elapsed time, not just "n rows." On top of that: calendar features (hour, day-of-week, month, weekend), cyclical encodings (`sin`/`cos`, removing the 11pm→0am discontinuity), configurable minute/day lags, and a 24h rolling mean/std computed after a 1-step shift so the target never leaks into its own rolling stats.

Lag correlation (`corr(speed, lag_X)`, all 125 segments, 4 months, 5-min grid):

| Lag | Correlation |
|---|---|
| 5 min | 0.969 |
| 10 min | 0.944 |
| 30 min | 0.896 |
| 1 day | 0.778 |
| 30 day | 0.671 |

Also built a **stuck-sensor diagnostic** that flags any run of ≥12 identical consecutive readings (1 hour on the 5-min grid) — almost certainly a frozen sensor rather than real traffic behavior, and something a status-code filter alone would miss. This caught 5.92% of all rows, with four segments stuck 100% of the time across the full 4 months.

**Takeaway:** short-horizon correlation (5–30 min) is high enough that any forecasting model has to beat a naive persistence baseline to be worth using. Longer horizons (1 day, 30 day) show meaningfully lower correlation, leaving real room for calendar/cyclical features to add value precisely where persistence weakens.
