---
title: "GTFS Route-Structure Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Parsed a GTFS feed with Pandas to compute route-level unique stop counts across multiple transit lines."
---

I worked through a GTFS problem set (3 routes, 16 stops, synthetic feed modeled on a generic downtown transit network — not Seoul-specific). What I finished:

| route_id | route_long_name         | n_stops |
| -------- | ----------------------- | ------- |
| R10      | Downtown ↔ SODO         | 5       |
| R12      | Magnolia ↔ Madison Park | 7       |
| R70      | Downtown ↔ Northgate    | 6       |

I got there by joining `stop_times` with `trips` to attach `route_id`, then counting unique `stop_id`s per route. I haven't gotten to the rest of the exercise yet — per-segment distance in a projected CRS (GeoPandas/Shapely), route length, average stop spacing, an interactive Folium map, and a 400m-walkshed accessibility bonus are still scaffolded, not implemented.
