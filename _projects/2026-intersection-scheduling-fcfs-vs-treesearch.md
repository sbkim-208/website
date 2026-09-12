---
title: "Cooperative Intersection Scheduling: FCFS vs. Tree Search"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A 7-algorithm benchmark of cooperative-driving schedulers against a first-come-first-served baseline at an unsignalized intersection."
thumbnail: /assets/img/projects/v2v-intersection-scheduling.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/v2v-intersection-coordinator
---

I built a browser-based simulator for an unsignalized intersection to answer a concrete question: can a cooperative scheduler beat first-come-first-served, and by how much once you actually measure it instead of assuming it? Vehicles spawn from 4 directions (Poisson arrivals), drive under an IDM car-following model, and — in V2V mode — request a time slot `{t0, t1}` from a pluggable scheduling algorithm before entering the conflict box. I wrote a Python server (`server.py`) that routes each request to the active algorithm and hot-reloads on file save, so I could swap in a new scheduler and re-benchmark without restarting the sim.

<img src="{{ '/assets/img/projects/v2v-intersection-scheduling.gif' | relative_url }}" alt="V2V intersection simulator running the MCTS+heuristic scheduler at 22 veh/min" style="max-width:100%;">

The MCTS+heuristic scheduler at 22 veh/min/approach — cyan lines are the live V2V mesh between vehicles inside comm range, amber vehicles are pacing to hit their assigned slot.

### Design decisions

**Yielding needs to be measured, not assumed.** My first version of the cost-benefit yielder (`algo_platoon`) used a threshold rule: yield whenever 2+ opposing vehicles were unassigned. Benchmarked, that made average delay **59% worse than FCFS** — almost every request chose to yield, and the yields chained into each other. I rewrote it to actually simulate both orderings ("I go first" vs. "the opposing platoon goes first") as real first-fit assignments and compare their total delay directly, picking whichever is lower. That version is the one in the benchmark below, at +28–31%. Same intuition, opposite result, because the second version measures the outcome instead of guessing at a proxy for it.

**Porting a research algorithm's rule literally doesn't preserve its intent.** Implementing Xu et al.'s Algorithm 2 (the MCTS rollout heuristic) against a single conflict box, I found the literal version degenerates to plain FIFO — nothing left to exploit once there's only one region vehicles can conflict in. The paper's advantage comes from letting non-conflicting vehicles cross in parallel, which only exists if the conflict zone is subdivided. I split the intersection into 4 conflict subzones (NE/NW/SE/SW quadrants of the crossing) so a vehicle occupies 2 subzones in sequence instead of the whole box at once — restoring the parallel-crossing case the rule is actually about, and with it the paper's claimed small-budget convergence advantage.

**A protocol assumption I almost missed.** Early benchmarks under my original "assign a slot immediately when requested" protocol showed MCTS tying with the much cheaper `algo_platoon` heuristic even at 60 vehicles — no sign of the global-search advantage the paper reports. Rather than conclude MCTS just wasn't worth it, I went back to the paper's protocol (IV-A): it re-plans every vehicle in the control zone every 2 seconds, instead of committing a slot the instant it's asked for. I implemented `replan()` to match, and MCTS's advantage over the cheaper heuristics only shows up once requests are evaluated together instead of one at a time — the gap was real, my protocol had just been hiding it.

### Algorithms

Seven schedulers share the same request payload (vehicle state, confirmed grants, per-lane last slot, full vehicle snapshot, in-range neighbors) and the same constraints (return only the requesting vehicle's slot, never overlap the opposing axis, same-lane headway is FIFO):

| Algorithm           | Approach                                                                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `algo_fcfs`         | FCFS baseline — first-fit into the earliest open gap by ETA                                                                                                                     |
| `algo_platoon`      | Cost-benefit yielding — evaluates "me first" vs. "let the opposing platoon go first" as two real assignments, picks the lower total delay                                       |
| `algo_priority`     | Non-preemptive emergency priority — reserves a virtual grant around an approaching EMS vehicle's window                                                                         |
| `algo_p2p`          | P2P approximation — only avoids grants heard within communication range (partial-visibility trade-off)                                                                          |
| `algo_mcts`         | Classical MCTS (UCB1 + random rollout, 0.05s / 400 iterations), adapted from Xu et al., _Cooperative Driving at Unsignalized Intersections Using Tree Search_, arXiv:1902.01024 |
| `algo_mcts_heur`    | MCTS with a subzone-decomposed rollout heuristic (4 conflict quadrants) in place of random rollout — faster convergence at small budgets                                        |
| `algo_p2p_priority` | MCTS + P2P visibility + weighted delay objective (EMS ×10, police ×5, bus ×2, general ×1)                                                                                       |

Benchmark (same scenario, sequential slot requests), average delay improvement over FCFS:

| Algorithm                      | Delay vs. FCFS | Note                           |
| ------------------------------ | -------------- | ------------------------------ |
| `algo_platoon`                 | +28–31%        | Near-zero compute cost         |
| `algo_mcts` / `algo_mcts_heur` | +30–33%        | ~50ms per request              |
| `algo_p2p_priority`            | +29%           | EMS delay 3.32s → 0.48s (−86%) |

One more finding that didn't come from a rewrite: priority policy and P2P visibility turned out to be independent axes. Priority works fine in a fully centralized scheduler — it's a scoring change, not a structural one — while P2P's partial visibility is a real safety-vs-decentralization trade-off I couldn't design away, only measure (`algo_p2p`'s comm-range slider trades SAFETY VIOLATIONS against decentralization directly in the sim).
