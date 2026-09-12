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

A* searching Gangnam → Hapjeong on the Seoul subway graph — frontier nodes shown with their f-scores as the search expands, followed by the built-in comparison table across all five algorithms.

### 2. Problem

I wanted to sharpen my interview-ready understanding of search algorithms on a real transportation network — not just implement BFS/DFS/Dijkstra/A*, but be able to say precisely why one beats another and by how much, on an actual graph instead of a whiteboard example.

### 3. Goals

I wanted every algorithm to share one interface, so adding a new one later wouldn't mean touching the frontend at all. And I didn't want to just assume A* would win because it's supposed to — I wanted to actually measure the payoff of each design decision I made, on the real ~50-station graph, not a toy one.

### 4. Process

Before writing `a_star.py` I considered a shortcut: precompute minimum transfer-to-transfer costs with Dijkstra once, then reuse that table as A*'s `h(n)` for every query. Working through it, I realized that table only lower-bounds the remaining cost for the specific pairs it was computed on — reusing it for an arbitrary query isn't automatically admissible, and an inadmissible heuristic silently breaks A*'s optimality guarantee. I used straight-line distance instead, which is a safe lower bound given travel-time-like edge weights and needs no precomputation at all.

I designed every algorithm to implement the same interface and yield `Step` frames, the one animation primitive the frontend needs to render any algorithm without knowing which one it is:

```python
@dataclass
class Step:
    iteration: int
    current: NodeId | None          # node currently being processed
    frontier: list[NodeId]          # nodes in queue / heap
    visited: list[NodeId]           # settled nodes
    distances: dict[NodeId, float]
    parent: dict[NodeId, NodeId]
    note: str                       # human-readable description
    done: bool
    path: list[NodeId]              # set when done
```

`core/registry.py` imports every module under `app/algorithms/` at startup, so a new algorithm shows up through `/algorithms` and the frontend dropdown automatically. That's the exact path I used to add A* after the first three algorithms were already running — no client-side changes needed.

The placeholder styling that looked fine on 8-node grid demos broke the moment I loaded the real ~50-station subway graph — labels overlapped and had too little contrast to read at a glance. I fixed that end to end: switched the font to Pretendard Variable for proper Korean + Latin glyph coverage, bumped label size from 11px to 13px, swapped translucent "glass" node backgrounds for solid state-tinted colors, raised edge opacity from 0.3 to 0.7, and scaled the subway layout's coordinates by 1.6× so adjacent stations stop crowding each other.

### 5. Result

I ran all five algorithms on the same Gangnam → Hapjeong route through `/eval`, specifically to check the A* payoff was real and not just assumed:

| Algorithm | Hops | Total Cost | Nodes Visited | Iterations |
|---|---:|---:|---:|---:|
| A* | 6 | 26 | **7** | 7 |
| DFS | 6 | 26 | 25 | 25 |
| BFS | 6 | 26 | 27 | 27 |
| Dijkstra | 6 | 26 | 40 | 40 |

All four reach the same optimal cost (26) and hop count (6) — there's no shortcut past these transfers — but they get there having looked at very different amounts of the graph. A* reaches it having visited only 7 of the ~50 stations, against Dijkstra's 40, because its heuristic keeps pulling the search toward Hapjeong instead of expanding outward in every direction evenly — the heuristic decision above actually paying off, measured rather than assumed. I don't think DFS's 25 means much beyond this one route, though — it's probably just this route's neighbor-list ordering happening to point roughly the right way, since nothing in DFS actually biases it toward the goal the way A*'s heuristic does.

### Architecture

| Layer           | Choice                                               |
| --------------- | ----------------------------------------------------- |
| Backend         | FastAPI + WebSocket (async step streaming)           |
| Frontend        | Vite + React + TypeScript                            |
| Graph rendering | React Flow (node/edge primitives, MiniMap, pan/zoom) |
| State           | Zustand                                              |

| Method | Path             | Purpose                                                         |
| ------ | ---------------- | ----------------------------------------------------------------- |
| `GET`  | `/algorithms`    | List registered algorithms                                      |
| `GET`  | `/graphs`        | List datasets                                                   |
| `GET`  | `/graphs/{name}` | Load a dataset's nodes + edges + layout                         |
| `POST` | `/eval`          | Run every algorithm on the same start/goal, return a comparison |
| `WS`   | `/ws/run`        | Stream `Step` frames for a single algorithm run                 |

Also registered, built while working through the session's LeetCode-style homework problems against the same `Step` contract: Number of Islands (`island_dfs`), Course Schedule (`dfs_topo`, cycle detection via topological sort), Shortest Path in Binary Matrix (`bfs_binary_matrix`, 8-directional), and Network Delay Time (`dijkstra_network_delay`), each with its own dataset (grids, a small dependency DAG, a weighted digraph).

I color each `Step` on the canvas as it streams in — gray (unvisited) → yellow (frontier, labeled with its running cost/score) → red (current) → green (visited) — with the final path drawn in blue with animated edges, so a search that's normally invisible inside a call stack is something anyone can actually watch happen, one settled node at a time. Backend correctness is checked with `pytest` smoke tests.
