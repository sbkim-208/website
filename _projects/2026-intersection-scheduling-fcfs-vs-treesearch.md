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

### 1. Overview

I built a browser-based simulator for an unsignalized intersection. Vehicles spawn from 4 directions (Poisson arrivals), drive under an IDM car-following model, and in V2V mode request a time slot `{t0, t1}` from a pluggable scheduling algorithm before entering the conflict box. Seven schedulers ended up sharing that request interface, benchmarked against a first-come-first-served baseline.

<img src="{{ '/assets/img/projects/v2v-intersection-scheduling.gif' | relative_url }}" alt="V2V intersection simulator running the MCTS+heuristic scheduler at 22 veh/min" style="max-width:100%;">

### 2. Problem

Cooperative scheduling beating first-come-first-served sounds obviously true. I wanted to know if it actually is, and by how much, once I measured it instead of assuming a smarter-sounding algorithm just wins.

### 3. Goals

My goal was to actually understand how cooperative driving works, not just read about it. So I went looking for a paper on how to improve over FCFS, found Xu et al.'s tree-search approach, and decided to implement it myself instead of just trusting the paper's numbers. That turned into a second goal I hadn't planned on: making sure my port of someone else's algorithm actually preserved what made it work in their setup, not just its surface-level rule.

### 4. Process

My first version of the cost-benefit yielder (`algo_platoon`) used a threshold rule: yield whenever 2+ opposing vehicles were unassigned. I benchmarked it before trusting it, and it made average delay 59% worse than FCFS, because almost every request chose to yield and the yields chained into each other. I rewrote it to actually simulate both orderings, "I go first" versus "the opposing platoon goes first", as real assignments, and compare their total delay directly.

Implementing Xu et al.'s Algorithm 2 against a single conflict box, I found the literal version just degenerates to plain FIFO. There's nothing left to exploit once there's only one region vehicles can conflict in. The paper's advantage comes from letting non-conflicting vehicles cross in parallel, which only exists if the conflict zone is subdivided, so I split the intersection into 4 conflict subzones (NE/NW/SE/SW) so a vehicle occupies 2 subzones in sequence instead of the whole box at once.

Even after that fix, early benchmarks under my original protocol, assigning a slot immediately when requested, showed MCTS basically tying with the much cheaper `algo_platoon` heuristic, even at 60 vehicles. Instead of concluding MCTS just wasn't worth it, I went back to the paper's actual protocol, which re-plans every vehicle in the control zone every 2 seconds instead of committing a slot the instant it's asked for. I implemented `replan()` to match it.

Seven schedulers ended up sharing the same request payload:

| Algorithm           | Approach                                                                                                                                                                        |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `algo_fcfs`         | FCFS baseline, first-fit into the earliest open gap by ETA                                                                                                                     |
| `algo_platoon`      | Cost-benefit yielding, compares "me first" vs. "let the opposing platoon go first" as two real assignments                                       |
| `algo_priority`     | Non-preemptive emergency priority, reserves a virtual grant around an approaching EMS vehicle's window                                                                         |
| `algo_p2p`          | P2P approximation, only avoids grants heard within communication range                                                                          |
| `algo_mcts`         | Classical MCTS (UCB1 + random rollout, 0.05s / 400 iterations), adapted from Xu et al., _Cooperative Driving at Unsignalized Intersections Using Tree Search_, arXiv:1902.01024 |
| `algo_mcts_heur`    | MCTS with a subzone-decomposed rollout heuristic in place of random rollout                                        |
| `algo_p2p_priority` | MCTS + P2P visibility + weighted delay objective (EMS ×10, police ×5, bus ×2, general ×1)                                                                                       |

### 5. Result

First, the yielding rewrite flipped the result completely, same idea, opposite outcome, because the second version measures the actual delay instead of guessing at a proxy for it:

| Algorithm                 | Delay vs. FCFS | Note                          |
| -------------------------- | -------------- | ------------------------------ |
| Threshold rule (v1)         | −59% (worse)   | Nearly every request yielded  |
| Cost-benefit rewrite (v2)   | +28–31%        | Same idea, measures the outcome |

Second, the subzone fix restored the parallel-crossing case the MCTS paper's advantage actually depends on. Without it I would have concluded the paper's algorithm just doesn't help here, which wasn't true. I'd just implemented a version of the intersection where its advantage couldn't exist.

Third, the protocol fix mattered just as much as the algorithm. Once requests were evaluated together every 2 seconds instead of one at a time, MCTS's advantage over the cheaper heuristics showed up. The gap was real, my protocol had just been hiding it:

| Algorithm                       | Delay vs. FCFS | Note                             |
| -------------------------------- | -------------- | --------------------------------- |
| `algo_platoon`                   | +28–31%        | Near-zero compute cost           |
| `algo_mcts` / `algo_mcts_heur`   | +30–33%        | ~50ms per request                |
| `algo_p2p_priority`              | +29%           | EMS delay 3.32s → 0.48s (−86%)   |

Fourth, a finding that didn't come from fixing a bug. Priority policy and P2P visibility turned out to be independent axes. Priority works fine in a fully centralized scheduler, since it's just a scoring change. P2P's partial visibility is a real safety-vs-decentralization trade-off, and I don't think it's something you can design away, only measure. The comm-range slider in the sim trades safety violations against decentralization directly.
