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

I built an interactive graph-algorithm simulator to sharpen my interview-ready understanding of search algorithms on real transportation networks — not just implement them, but be able to say precisely why one beats another and by how much. A FastAPI + WebSocket backend runs a search algorithm step by step and streams each intermediate state to a React/TypeScript frontend I wrote to render it live on a node-link canvas.

<img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search animating over the Seoul subway graph, then an algorithm-comparison table" style="max-width:100%;">

A* searching Gangnam → Hapjeong on the Seoul subway graph — frontier nodes shown with their f-scores as the search expands, followed by the built-in comparison table across all five algorithms.

### Design decisions

**Why A\* over a precomputed lookup table.** Before writing `a_star.py`, I considered a shortcut: precompute minimum transfer-to-transfer costs with Dijkstra once, then reuse that table as A*'s `h(n)` for every query. I rejected it once I worked through the guarantee it would break — the table only lower-bounds the remaining cost for the specific pairs it was computed on, so reusing it for an arbitrary query isn't automatically admissible, and an inadmissible heuristic silently voids A*'s optimality guarantee. I used straight-line distance instead: a safe lower bound given travel-time-like edge weights and a layout that tracks true geography, and it needs no precomputation step at all.

**Making ~50 overlapping Korean labels legible.** The placeholder styling that looked fine on 8-node grid demos broke the moment I loaded the real ~50-station subway graph: labels overlapped and had too little contrast to read at a glance. I fixed it end to end — switched the font to Pretendard Variable for proper Korean + Latin glyph coverage, bumped label size from 11px to 13px, swapped translucent "glass" node backgrounds for solid state-tinted colors, raised edge opacity from 0.3 to 0.7, and scaled the subway layout's coordinates by 1.6× so adjacent stations stop crowding each other.

### Architecture

I designed every algorithm to implement the same interface and yield `Step` frames — the one animation primitive the frontend needs to render any algorithm without knowing which one it is:

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

I built `core/registry.py` to import every module under `app/algorithms/` at startup, so a new algorithm — e.g. `a_star.py` with a `@register`-decorated class — shows up through `/algorithms` and the frontend dropdown automatically, with zero client-side changes. That's not a hypothetical: it's the exact path I used to add A* after the first three algorithms were already running.

| Layer           | Choice                                               |
| --------------- | ---------------------------------------------------- |
| Backend         | FastAPI + WebSocket (async step streaming)           |
| Frontend        | Vite + React + TypeScript                            |
| Graph rendering | React Flow (node/edge primitives, MiniMap, pan/zoom) |
| State           | Zustand                                              |

| Method | Path             | Purpose                                                         |
| ------ | ---------------- | --------------------------------------------------------------- |
| `GET`  | `/algorithms`    | List registered algorithms                                      |
| `GET`  | `/graphs`        | List datasets                                                   |
| `GET`  | `/graphs/{name}` | Load a dataset's nodes + edges + layout                         |
| `POST` | `/eval`          | Run every algorithm on the same start/goal, return a comparison |
| `WS`   | `/ws/run`        | Stream `Step` frames for a single algorithm run                 |

### Algorithms

| Algorithm | Implementation |
|---|---|
| BFS | Unweighted shortest path — queue-based, hop count |
| DFS | Iterative, stack-based traversal |
| Dijkstra | `heapq`-based, weighted, early-exits once the goal is popped |
| A* | `g(n) + h(n)` with a straight-line-distance heuristic |

Also registered, built against the same `Step` contract while working through the session's LeetCode-style homework problems: **Number of Islands** (`island_dfs`), **Course Schedule** (`dfs_topo`, cycle detection via topological sort), **Shortest Path in Binary Matrix** (`bfs_binary_matrix`, 8-directional), and **Network Delay Time** (`dijkstra_network_delay`) — each with its own dataset (grids, a small dependency DAG, a weighted digraph).

The `/eval` endpoint runs every registered algorithm against the same start/goal pair and reports path, hop count, path cost, nodes visited, iterations, elapsed time, and whether the result matches Dijkstra's cost (`optimal: true/false`) — a direct, on-graph comparison of correctness and search efficiency, not just a single run.

### Case study: Gangnam → Hapjeong

I ran all five algorithms on this route through `/eval` to check the A* payoff was real, not assumed:

| Algorithm | Hops | Total Cost | Nodes Visited | Iterations |
|---|---:|---:|---:|---:|
| A* | 6 | 26 | **7** | 7 |
| DFS | 6 | 26 | 25 | 25 |
| BFS | 6 | 26 | 27 | 27 |
| Dijkstra | 6 | 26 | 40 | 40 |

All four reach the same optimal cost (26) and hop count (6) — there's no shortcut past these transfers — but they get there having looked at very different amounts of the graph. A* reaches it having visited only 7 of the ~50 stations, against Dijkstra's 40, because its heuristic keeps pulling the search toward Hapjeong instead of expanding uniformly outward in every direction — the design decision above paying off, measured rather than assumed. DFS's 25 isn't evidence it's competitive with A* in general — it's an artifact of this route's neighbor-list ordering happening to point roughly the right way; nothing in DFS biases it toward the goal the way A*'s heuristic does.

### Datasets

| Dataset | Notes |
|---|---|
| 6×8 grid with obstacles | Number-of-Islands / binary-matrix style |
| 6×10 open grid | Fully connected |
| Binary matrix (8×6, 8-directional) | Shortest Path in Binary Matrix |
| Course schedule (with / without a cycle) | Small dependency DAG for topological-sort cycle detection |
| Network delay (weighted digraph) | Network Delay Time |
| Seoul subway (~50 stations) | Lines 1–5 plus highlights of 6/7/9, edges weighted by approximate inter-station travel time in minutes; same-named stations across lines share one node, so transfers are implicit in the graph rather than a separate edge type |

### Visualization

I color each `Step` on the canvas as it streams in: gray (unvisited) → yellow (frontier, labeled with its running cost/score) → red (current) → green (visited), with the final path drawn in blue with animated edges — so a search that's normally invisible inside a call stack is something anyone can watch happen, one settled node at a time. I check backend correctness with `pytest` smoke tests.
