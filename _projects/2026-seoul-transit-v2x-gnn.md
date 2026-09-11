---
title: "Seoul Transit Network Analysis & GNN-Based V2X Scheduling"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Graph algorithms over Seoul's GTFS/GIS transit network, plus a GNN-based V2X scheduler benchmarked against FCFS on latency, deadline compliance, throughput, and fairness."
---

This project combines graph algorithms with GTFS and GIS data to analyze connectivity, accessibility, and routing across Seoul's transit network. It then evaluates whether GNN-based V2X scheduling can outperform FCFS in latency, deadline compliance, throughput, and fairness under dynamic operating conditions.

- Modeled Seoul transit stations as an adjacency-list graph and implemented BFS, DFS, and heap-based Dijkstra routing to analyze connectivity and shortest paths.
- Parsed GTFS feeds and used GeoPandas and Shapely to calculate route coverage, stop density, interstation distances, and station accessibility; generated interactive Folium maps.
- Evaluated FCFS-based V2X scheduling under dynamic traffic conditions, variable network loads, and competing vehicle priorities to identify latency, deadline compliance, throughput, and fairness limitations.
- Developed a small-scale GNN scheduler that represents vehicles and communication links as a graph and compares performance against FCFS using latency, miss rate, throughput, and fairness.
