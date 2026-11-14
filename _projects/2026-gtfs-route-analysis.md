---
title: "GTFS Route-Structure Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I used Pandas to count unique stops on three routes in a synthetic GTFS feed. The main work was joining the tables without counting the same stop more than once."
---

How many stops does each route serve? That sounded like a small question, but the answer was spread across the GTFS tables. I worked through this exercise to see whether I could get it from the raw records myself.

## Three routes, sixteen stops
{: #study-scope }

The input was a synthetic downtown transit feed with 3 routes and 16 stops. It wasn't a Seoul dataset, and the route names below are labels from the exercise.

A stop can appear in several trips and belong to more than one route. Counting rows in `stop_times` would count repeated visits as additional stops.

## Joining the records
{: #study-process }

In Pandas, I joined `stop_times` with `trips` to attach a `route_id` to each stop-time record. Then I grouped by `route_id` and counted distinct `stop_id` values within each group. The part I had underestimated was deciding what to count before doing the calculation.

## Stops served by each route
{: #study-results }

| Route ID | Route name | Unique stops |
| --- | --- | ---: |
| R10 | Downtown ↔ SODO | 5 |
| R12 | Magnolia ↔ Madison Park | 7 |
| R70 | Downtown ↔ Northgate | 6 |

These counts overlap when routes share a stop, so adding them would not give the number of distinct stops in the feed. They also say nothing about route length, stop spacing, or coverage on a real transit system.

## What I haven't finished
{: #study-next }

That's as far as I've gotten. The distance, spacing, map, and walking-accessibility parts are still scaffolded. To continue, I'd calculate segment and route lengths in a projected coordinate system, measure spacing along each route's stop sequence, and map the routes and stops in Folium. The optional 400 m walkshed exercise is also unfinished.
