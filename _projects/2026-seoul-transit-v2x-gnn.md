---
title: "Cooperative Intersection Scheduling: FCFS vs. Tree Search"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A 7-algorithm benchmark of cooperative-driving schedulers against a first-come-first-served baseline at an unsignalized intersection, a GTFS route-structure exercise, and a from-scratch LSTM/GCN comparison against RF and MLP baselines for transportation forecasting."
---

Three self-contained exercises: a benchmark of cooperative-driving scheduling algorithms against a first-come-first-served (FCFS) baseline at an unsignalized intersection, a GTFS route-structure analysis, and from-scratch LSTM/GCN implementations benchmarked against classical baselines.

### 1. GTFS route-structure analysis

Worked through a GTFS problem set (3 routes, 16 stops, synthetic feed modeled on a generic downtown transit network — not Seoul-specific). Completed:

| route_id | route_long_name | n_stops |
|---|---|---|
| R10 | Downtown ↔ SODO | 5 |
| R12 | Magnolia ↔ Madison Park | 7 |
| R70 | Downtown ↔ Northgate | 6 |

Computed by joining `stop_times` with `trips` to attach `route_id`, then counting unique `stop_id`s per route. The remaining parts of the exercise — per-segment distance in a projected CRS (GeoPandas/Shapely), route length, average stop spacing, an interactive Folium map, and a 400m-walkshed accessibility bonus — are scaffolded but not yet completed.

### 2. Cooperative intersection scheduling: FCFS vs. tree search

Built a browser-based simulator for an unsignalized intersection: vehicles spawn from 4 directions (Poisson arrivals), drive under an IDM car-following model, and — in V2V mode — request a time slot `{t0, t1}` from a pluggable scheduling algorithm before entering the conflict box. A Python server (`server.py`) routes each request to the active algorithm and hot-reloads on file save.

Seven schedulers share the same request payload (vehicle state, confirmed grants, per-lane last slot, full vehicle snapshot, in-range neighbors) and the same constraints (return only the requesting vehicle's slot, never overlap the opposing axis, same-lane headway is FIFO):

| Algorithm | Approach |
|---|---|
| `algo_fcfs` | FCFS baseline — first-fit into the earliest open gap by ETA |
| `algo_platoon` | Cost-benefit yielding — evaluates "me first" vs. "let the opposing platoon go first" as two real assignments, picks the lower total delay |
| `algo_priority` | Non-preemptive emergency priority — reserves a virtual grant around an approaching EMS vehicle's window |
| `algo_p2p` | P2P approximation — only avoids grants heard within communication range (partial-visibility trade-off) |
| `algo_mcts` | Classical MCTS (UCB1 + random rollout, 0.05s / 400 iterations), adapted from Xu et al., *Cooperative Driving at Unsignalized Intersections Using Tree Search*, arXiv:1902.01024 |
| `algo_mcts_heur` | MCTS with a lane-batching rollout heuristic in place of random rollout — faster convergence at small budgets |
| `algo_p2p_priority` | MCTS + P2P visibility + weighted delay objective (EMS ×10, police ×5, bus ×2, general ×1) |

Benchmark (same scenario, sequential slot requests), average delay improvement over FCFS:

| Algorithm | Delay vs. FCFS | Note |
|---|---|---|
| `algo_platoon` | +28–31% | Near-zero compute cost |
| `algo_mcts` / `algo_mcts_heur` | +30–33% | ~50ms per request |
| `algo_p2p_priority` | +29% | EMS delay 3.32s → 0.48s (−86%) |

**What actually moved the needle:**
1. Priority policy and P2P visibility are independent axes — priority works fine in a centralized scheduler, while P2P's partial visibility is a real safety-vs-decentralization trade-off, not a free upgrade.
2. Heuristic *shape* matters more than the idea behind it: a threshold-based version of the same lane-batching idea made things 59% *worse*, while an outcome-evaluated version improved delay 29% — same intuition, opposite result.
3. Under an "assign immediately on request" protocol, MCTS's global-search advantage doesn't show up — it ties with the much cheaper `platoon` heuristic even at 60 vehicles. The paper's advantage requires a 2-second batched-replanning protocol instead.
4. Porting the paper's Algorithm 2 rule 2 literally to a single conflict box degenerates to plain FIFO — reinterpreting its intent (parallel-crossing exploitation) as axis-batching was necessary to reproduce the claimed small-budget convergence advantage.

### 3. LSTM and GCN from scratch, benchmarked against classical baselines

Two small PyTorch models, each built from scratch (no `torch_geometric`) and evaluated against a non-deep-learning baseline on the same task and split, rather than in isolation.

**Sequence forecasting — LSTM vs. Random Forest.** A single-layer LSTM (`hidden=32`) predicted next-hour bike-share demand from a 24-hour sliding window, with no hand-built lag/calendar features — the model has to learn the daily cycle purely from the raw sequence. A Random Forest on the same windows, flattened into 24 lag features, served as the classical baseline:

| Model | MAE | RMSE | R² |
|---|---|---|---|
| LSTM (raw sequence, no lag features) | 9.76 | 14.88 | 0.884 |
| Random Forest (24 flattened lag features) | 5.04 | 8.19 | **0.965** |

RF wins here — with a few months of hourly data, hand-built lag features plus a tree ensemble still beat a from-scratch LSTM. A follow-up hidden-size sweep (`[8, 16, 32, 64]`, test RMSE) checked whether the LSTM was simply under-capacity rather than fundamentally the wrong tool for this data size.

**Node prediction — GCN vs. MLP.** On a 16-node synthetic transit graph (main line + 2 branches + a cross-connection), the ridership target was generated to depend on a station's *neighbors'* transfer-count and parking values, not only its own — a case an MLP structurally can't solve. A 2-layer GCN (`H' = ReLU(Â·H·W)`) and a plain MLP were trained on the same 8 labeled nodes and evaluated on the other 8:

| Model | MAE | R² |
|---|---|---|
| MLP (no graph) | 105.94 | **−4.25** |
| GCN (uses `Â`) | 27.95 | **0.733** |

The MLP does worse than predicting the mean because it never sees neighbor information; the GCN propagates it through the normalized adjacency matrix and fits well. Together the two experiments make the same point from opposite sides: architecture choice only pays off when it matches what the data actually depends on — sequence order for the LSTM case (where RF's explicit lag features already captured that dependence better), and neighbor structure for the GCN case (where nothing but the graph could have captured it).
