---
title: "Seoul Station Graph Routing Visualizer"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A plug-and-play graph-algorithm playground — FastAPI + WebSocket backend, React/TypeScript frontend — that streams BFS, DFS, Dijkstra, and A* step-by-step over a ~50-station Seoul subway graph."
thumbnail: /assets/img/projects/seoul-graph-routing.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/session-4-graph-playground
---

### 1. Overview

I built an interactive graph-algorithm simulator over a ~50-station Seoul subway graph. A FastAPI + WebSocket backend runs a search algorithm step by step and streams each intermediate state to a React/TypeScript frontend I wrote to render it live on a node-link canvas.

<img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search animating over the Seoul subway graph, then an algorithm-comparison table" style="max-width:100%;">

### 2. Problem

I wanted to sharpen my understanding of search algorithms on a real transportation network, not just implement BFS, DFS, Dijkstra, and A*, but actually be able to say why one beats another and by how much, on a real graph instead of a whiteboard example.

### 3. Goals

I wanted every algorithm to share one interface, so adding a new one later wouldn't mean touching the frontend at all. And I didn't want to just assume A* would win because it's supposed to. I wanted to actually measure the payoff of each design decision, on the real ~50-station graph, not a toy one.

### 4. Process

Before writing `a_star.py` I looked at a shortcut: precompute costs with Dijkstra once and reuse that table as A*'s heuristic. It turned out that isn't safe for an arbitrary query, since the table only lower-bounds the cost for the specific pairs it was built from, so I used straight-line distance instead, which needs no precomputation at all.

Every algorithm shares one interface and streams step-by-step frames, so the frontend can render any of them without knowing which one it is. A new algorithm gets auto-discovered at startup, which is how I added A* later without touching the frontend at all.

The placeholder styling that looked fine on small demo graphs broke once I loaded the real ~50-station graph. Labels overlapped and were hard to read, so I fixed the font, label size, node colors, and layout spacing until it held up.

Backend is FastAPI + WebSocket, frontend is React + TypeScript with React Flow for the graph canvas. `/eval` runs every algorithm on the same start/goal and compares them; `/ws/run` streams a single run. Algorithms: BFS, DFS, Dijkstra, and A*, plus a few more I added while working through the session's homework problems against the same interface (Number of Islands, Course Schedule, Shortest Path in Binary Matrix, Network Delay Time).

### 5. Result

I ran all five algorithms on the same Gangnam → Hapjeong route through `/eval`, to check the A* payoff was real and not just assumed:

| Algorithm | Hops | Total Cost | Nodes Visited | Iterations |
|---|---:|---:|---:|---:|
| A* | 6 | 26 | **7** | 7 |
| DFS | 6 | 26 | 25 | 25 |
| BFS | 6 | 26 | 27 | 27 |
| Dijkstra | 6 | 26 | 40 | 40 |

All four reach the same optimal cost and hop count, so there's no shortcut past these transfers, but they get there having looked at very different amounts of the graph. A* visited only 7 of the ~50 stations, against Dijkstra's 40, because the heuristic keeps pulling the search toward Hapjeong instead of expanding outward evenly. I don't think DFS's 25 means much beyond this one route though. It's probably just this route's neighbor-list ordering happening to point roughly the right way, since nothing in DFS actually biases it toward the goal.
