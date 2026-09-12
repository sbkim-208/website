---
title: "GTFS Route-Structure Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Parsed a GTFS feed with Pandas to compute route-level unique stop counts across multiple transit lines."
---

### 1. Overview

I worked through a GTFS problem set — 3 routes, 16 stops, a synthetic feed modeled on a generic downtown transit network, not Seoul-specific — to characterize route structure directly from the raw feed instead of trusting a route map at face value.

### 2. Problem

A GTFS feed technically contains everything you'd need to describe a route — stops, trip sequences, geometry — but none of it is handed to you pre-joined. The problem set was really about whether I could actually pull a basic structural fact (how many stops does each route serve) out of the raw tables myself.

### 3. Goals

The immediate goal was just unique stop counts per route. The problem set goes further than that though — per-segment distance in a projected CRS, route length, average stop spacing, an interactive Folium map, and a 400m-walkshed accessibility bonus — and I haven't gotten to any of that part yet.

### 4. Process

I joined `stop_times` with `trips` to attach `route_id` to each stop time, then counted unique `stop_id`s per route.

### 5. Result

| route_id | route_long_name         | n_stops |
| -------- | ------------------------ | ------- |
| R10      | Downtown ↔ SODO          | 5       |
| R12      | Magnolia ↔ Madison Park  | 7       |
| R70      | Downtown ↔ Northgate     | 6       |

That's as far as I've gotten — the distance/spacing/map/walkshed parts are still scaffolded, not implemented.
