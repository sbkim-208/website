---
title: "NYC Traffic Sensor Network: Reliability & Congestion Analysis"
category: other            # not "research" -> appears under "Other Projects"
year: 2026
summary: "Reliability and congestion analysis of 4.2M NYC DOT traffic sensor readings across 5 boroughs, uncovering why sensors fail and how congestion actually differs by borough."
thumbnail: /assets/img/projects/nyc-traffic-borough-reliability.png
links:
  - name: Code
    url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

Analyzed 4.2M rows of NYC DOT traffic speed sensor data (April–August 2024, 5 boroughs) to answer two questions: how reliable are the sensors, and how does congestion actually vary across the city?

Sensor reliability varies sharply by borough (56% in Manhattan vs. 93% in Queens) — not because congestion wears sensors down, a hypothesis I tested directly on the data and rejected, but because valid-reading availability clusters around specific times of day per segment. After cleaning (removing 16 fully-dead segments, time-based interpolation for the rest), a time-series analysis showed Manhattan is uniformly slow all day rather than "worst at rush hour," while Brooklyn, Bronx, and Queens are highway-fast overnight and collapse hardest during peak hours. Also built lag/rolling features for downstream ML forecasting, including a stuck-sensor diagnostic that catches frozen readings a status-code filter alone would miss.
