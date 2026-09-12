---
title: "GTFS Route-Structure Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Parsed a GTFS feed with Pandas to compute route-level unique stop counts across multiple transit lines."
---

Worked through a GTFS problem set (3 routes, 16 stops, synthetic feed modeled on a generic downtown transit network — not Seoul-specific). Completed:

| route_id | route_long_name         | n_stops |
| -------- | ----------------------- | ------- |
| R10      | Downtown ↔ SODO         | 5       |
| R12      | Magnolia ↔ Madison Park | 7       |
| R70      | Downtown ↔ Northgate    | 6       |

Computed by joining `stop_times` with `trips` to attach `route_id`, then counting unique `stop_id`s per route. The remaining parts of the exercise — per-segment distance in a projected CRS (GeoPandas/Shapely), route length, average stop spacing, an interactive Folium map, and a 400m-walkshed accessibility bonus — are scaffolded but not yet completed.
