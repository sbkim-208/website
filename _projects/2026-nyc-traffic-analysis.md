---
title: "NYC Traffic Sensor Network: Reliability & Congestion Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Reliability and congestion analysis of 4.2M NYC DOT traffic sensor readings across 5 boroughs, uncovering why sensors fail and how congestion actually differs by borough."
thumbnail: /assets/img/projects/nyc-traffic-borough-reliability.png
links:
    - name: Code
      url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

Analyzed 4.2M rows of NYC DOT traffic speed sensor data (April–August 2024, 5 boroughs) to answer two questions: how reliable are the sensors, and how does congestion actually vary across the city?

Sensor reliability varies sharply by borough (56% in Manhattan vs. 93% in Queens). I directly tested the obvious hypothesis — that heavy traffic wears sensors down on congested corridors — on the two partially-dead Manhattan segments that still had some valid readings. The result was the opposite of what "congestion breaks sensors" predicts: error rate was *lowest* during rush hour (72–92%) and *highest* overnight (~100%). So congestion doesn't explain sensor failure; what actually varies is which hours of the day a given segment happens to report valid data at all, and that window has to be checked per segment rather than assumed.

After cleaning (removing 16 fully-dead segments, time-based interpolation for the rest), a time-series analysis showed Manhattan is uniformly slow all day rather than "worst at rush hour," while Brooklyn, Bronx, and Queens are highway-fast overnight and collapse hardest during peak hours — they carry the network's actual peak-hour congestion problem.

Also built lag/rolling features for downstream ML forecasting, including a stuck-sensor diagnostic (12+ identical consecutive readings) that flags frozen sensors a status-code filter alone would miss — 5.92% of all rows, with four segments stuck 100% of the time across the full 4 months. Lag correlation stays very high at short horizons (0.97 at 5 min, 0.90 at 30 min) but drops off meaningfully at longer ones (0.78 at 1 day, 0.67 at 30 day): any forecasting model has to beat a naive persistence baseline to be worth using at short horizons, while calendar/cyclical features have real room to add value precisely where persistence weakens, at the day-plus horizon.
