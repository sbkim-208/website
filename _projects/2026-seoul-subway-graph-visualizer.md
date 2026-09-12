---
title: "Seoul Station Graph Routing Visualizer"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A plug-and-play graph-algorithm playground — FastAPI + WebSocket backend, React/TypeScript frontend — that streams BFS, DFS, and Dijkstra step-by-step over a ~50-station Seoul subway graph."
links:
    - name: Code
      url: https://github.com/sbkim-208/session-4-graph-playground
---

An interactive graph-algorithm simulator built for traffic-engineering interview prep: a FastAPI + WebSocket backend runs BFS, DFS, and Dijkstra step by step and streams each intermediate state to a React/TypeScript frontend, which renders it live on a node-link canvas.

### Architecture

Every algorithm implements the same interface and yields `Step` frames — the one animation primitive the frontend needs to render any algorithm without knowing which one it is:

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

`core/registry.py` imports every module under `app/algorithms/` at startup, so a new algorithm — e.g. dropping in `a_star.py` with a `@register`-decorated class — is exposed through `/algorithms` and the frontend dropdown automatically, no client-side change needed.

| Layer | Choice |
|---|---|
| Backend | FastAPI + WebSocket (async step streaming) |
| Frontend | Vite + React + TypeScript |
| Graph rendering | React Flow (node/edge primitives, MiniMap, pan/zoom) |
| State | Zustand |

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/algorithms` | List registered algorithms |
| `GET` | `/graphs` | List datasets |
| `GET` | `/graphs/{name}` | Load a dataset's nodes + edges + layout |
| `POST` | `/eval` | Run every algorithm on the same start/goal, return a comparison |
| `WS` | `/ws/run` | Stream `Step` frames for a single algorithm run |

### Algorithms

| Algorithm | Implementation |
|---|---|
| BFS | Unweighted shortest path — queue-based, hop count |
| DFS | Iterative, stack-based traversal |
| Dijkstra | `heapq`-based, weighted, early-exits once the goal is popped |
| A* | Designed as the next plug-and-play addition — `g(n) + h(n)` — not yet implemented as a registered algorithm |

The `/eval` endpoint runs every registered algorithm against the same start/goal pair and reports path, hop count, path cost, nodes visited, iterations, elapsed time, and whether the result matches Dijkstra's cost (`optimal: true/false`) — a direct, on-graph illustration of why BFS's fewest-hops answer isn't always Dijkstra's least-cost answer once edges are weighted.

### Case study: Gangnam → Hapjeong

Running BFS, DFS, and Dijkstra on this same-line route through `/eval`:

| Algorithm | Hops | Total Cost | Nodes Visited | Iterations |
|---|---:|---:|---:|---:|
| DFS | 6 | 26 | 25 | 25 |
| BFS | 6 | 26 | 27 | 27 |
| Dijkstra | 6 | 26 | 40 | 40 |

All three land on the same path and cost — expected on a single line with uniform per-segment weights — but DFS touches the fewest nodes here. That's incidental to this particular graph and neighbor ordering, not a general property of DFS, which is what motivated adding A* to the design: it uses `g(n) + h(n)` (accumulated cost + estimated remaining cost) to bias search toward the goal, and can reach the same optimal cost while exploring fewer nodes than Dijkstra, given an admissible heuristic.

One design considered was precomputing minimum transfer-to-transfer costs with Dijkstra and feeding them into A* as `h(n)` for real-time queries. The catch: a table of transfer-station-to-transfer-station distances isn't automatically a valid heuristic for an arbitrary station pair — it only lower-bounds the remaining cost correctly once composed with the querying station's actual distance to its nearest relevant transfer node. Getting this wrong (over-estimating the remainder) breaks A*'s optimality guarantee, and a heuristic that isn't consistent forces reopening closed nodes in graph-search A* — the standard implementation this project uses assumes it won't need to.

### Datasets

| Dataset | Notes |
|---|---|
| 6×8 grid with obstacles | Number-of-Islands / binary-matrix style |
| 6×10 open grid | Fully connected |
| Seoul subway (~50 stations) | Lines 1–5 plus highlights of 6/7/9, edges weighted by approximate inter-station travel time in minutes; same-named stations across lines share one node, so transfers are implicit in the graph rather than a separate edge type |

### Visualization

Each `Step` is colored on the canvas as it streams in: gray (unvisited) → yellow (frontier) → red (current) → green (visited), with the final path drawn in blue with animated edges. Backend correctness is checked with `pytest` smoke tests.
