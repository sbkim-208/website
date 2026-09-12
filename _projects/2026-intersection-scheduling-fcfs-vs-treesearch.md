---
title: "Cooperative Intersection Scheduling: FCFS vs. Tree Search"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A 7-algorithm benchmark of cooperative-driving schedulers against a first-come-first-served baseline at an unsignalized intersection."
---

Built a browser-based simulator for an unsignalized intersection: vehicles spawn from 4 directions (Poisson arrivals), drive under an IDM car-following model, and — in V2V mode — request a time slot `{t0, t1}` from a pluggable scheduling algorithm before entering the conflict box. A Python server (`server.py`) routes each request to the active algorithm and hot-reloads on file save.

Seven schedulers share the same request payload (vehicle state, confirmed grants, per-lane last slot, full vehicle snapshot, in-range neighbors) and the same constraints (return only the requesting vehicle's slot, never overlap the opposing axis, same-lane headway is FIFO):

| Algorithm           | Approach                                                                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `algo_fcfs`         | FCFS baseline — first-fit into the earliest open gap by ETA                                                                                                                     |
| `algo_platoon`      | Cost-benefit yielding — evaluates "me first" vs. "let the opposing platoon go first" as two real assignments, picks the lower total delay                                       |
| `algo_priority`     | Non-preemptive emergency priority — reserves a virtual grant around an approaching EMS vehicle's window                                                                         |
| `algo_p2p`          | P2P approximation — only avoids grants heard within communication range (partial-visibility trade-off)                                                                          |
| `algo_mcts`         | Classical MCTS (UCB1 + random rollout, 0.05s / 400 iterations), adapted from Xu et al., _Cooperative Driving at Unsignalized Intersections Using Tree Search_, arXiv:1902.01024 |
| `algo_mcts_heur`    | MCTS with a lane-batching rollout heuristic in place of random rollout — faster convergence at small budgets                                                                    |
| `algo_p2p_priority` | MCTS + P2P visibility + weighted delay objective (EMS ×10, police ×5, bus ×2, general ×1)                                                                                       |

Benchmark (same scenario, sequential slot requests), average delay improvement over FCFS:

| Algorithm                      | Delay vs. FCFS | Note                           |
| ------------------------------ | -------------- | ------------------------------ |
| `algo_platoon`                 | +28–31%        | Near-zero compute cost         |
| `algo_mcts` / `algo_mcts_heur` | +30–33%        | ~50ms per request              |
| `algo_p2p_priority`            | +29%           | EMS delay 3.32s → 0.48s (−86%) |

**What actually moved the needle:**

1. Priority policy and P2P visibility are independent axes — priority works fine in a centralized scheduler, while P2P's partial visibility is a real safety-vs-decentralization trade-off, not a free upgrade.
2. Heuristic _shape_ matters more than the idea behind it: a threshold-based version of the same lane-batching idea made things 59% _worse_, while an outcome-evaluated version improved delay 29% — same intuition, opposite result.
3. Under an "assign immediately on request" protocol, MCTS's global-search advantage doesn't show up — it ties with the much cheaper `platoon` heuristic even at 60 vehicles. The paper's advantage requires a 2-second batched-replanning protocol instead.
4. Porting the paper's Algorithm 2 rule 2 literally to a single conflict box degenerates to plain FIFO — reinterpreting its intent (parallel-crossing exploitation) as axis-batching was necessary to reproduce the claimed small-budget convergence advantage.
