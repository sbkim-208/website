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

I built an interactive graph-algorithm simulator to prepare for traffic-engineering interviews: a FastAPI + WebSocket backend runs a search algorithm step by step and streams each intermediate state to a React/TypeScript frontend I wrote to render it live on a node-link canvas.

<img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search animating over the Seoul subway graph, then an algorithm-comparison table" style="max-width:100%;">

A* searching Gangnam → Hapjeong on the Seoul subway graph — frontier nodes shown with their f-scores as the search expands, followed by the built-in comparison table across all five algorithms.

### Design decisions

**Why A\* over a precomputed lookup table.** Before writing `a_star.py`, I considered speeding up repeated queries by precomputing minimum transfer-to-transfer costs with Dijkstra once, then reusing that table as A*'s `h(n)` for any station pair. I rejected it: the table only lower-bounds the remaining cost correctly for the specific pairs it was actually computed on, so reusing it for an arbitrary query isn't automatically admissible — and an inadmissible heuristic breaks A*'s optimality guarantee. I used straight-line distance instead. It's a safe lower bound here because edge weights are travel-time-like and the layout roughly tracks true geography, and unlike the lookup table it needs no precomputation step at all.

**Making ~50 overlapping Korean labels legible.** The placeholder node styling I'd built for the small 8-node grid demos fell apart the moment I loaded the real ~50-station subway graph — labels overlapped each other and had too little contrast against the dark canvas to read at a glance. I fixed it by switching the font to Pretendard Variable (proper Korean + Latin glyph coverage), bumping label size from 11px to 13px, replacing translucent "glass" node backgrounds with solid state-tinted colors, raising edge opacity from 0.3 to 0.7, and scaling the subway layout's coordinates by 1.6× so adjacent stations stop crowding each other.

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

I built `core/registry.py` to import every module under `app/algorithms/` at startup, so I can drop in a new algorithm — e.g. `a_star.py` with a `@register`-decorated class — and have it show up through `/algorithms` and the frontend dropdown automatically, with no client-side change. This is the interface I actually used to add A* after the first three algorithms were already running.

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

I color each `Step` on the canvas as it streams in: gray (unvisited) → yellow (frontier, labeled with its running cost/score) → red (current) → green (visited), with the final path drawn in blue with animated edges. I check backend correctness with `pytest` smoke tests.
