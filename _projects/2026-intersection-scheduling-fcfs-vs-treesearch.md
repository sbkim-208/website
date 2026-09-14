---
title: "Cooperative Intersection Scheduling: FCFS vs. Tree Search"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "FIFO vs. MCTS vs. a physics-aware MCTS at an unsignalized intersection, measured instead of assumed."
thumbnail: /assets/img/projects/v2v-intersection-scheduling.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/v2v-intersection-coordinator
---

### 1. Overview

I built a browser-based simulator for an unsignalized intersection. Vehicles spawn from 4 directions (Poisson arrivals), drive under an IDM car-following model, and in V2V mode request a time slot `{t0, t1}` from a pluggable scheduling algorithm before entering the conflict box. I ended up comparing three schedulers that share that request interface: a FIFO baseline, the tree-search algorithm from Xu et al., and my own extension of it.

<img src="{{ '/assets/img/projects/v2v-intersection-scheduling.gif' | relative_url }}" alt="V2V intersection simulator running the MCTS+heuristic scheduler at 22 veh/min" style="max-width:100%;">

### 2. Problem

Cooperative scheduling beating first-come-first-served sounds obviously true. I wanted to know if it actually is, and by how much, once I measured it instead of assuming a smarter-sounding algorithm just wins.

### 3. Goals

My goal was to better understand how connected vehicles cooperate with each other at unsignalized intersections, starting from how FIFO-based coordination works. Beyond that, I wanted to explore whether different approach could reduce delay, so I looked for a paper on the problem and found Xu et al.'s tree-search approach. I decided to implement it myself to understand how it worked and evaluate it in my own simulator. 



### 4. Process

I first read Xu et al.'s paper to understand how MCTS searches for a passing order with low total delay, and how heuristic rules help that search. 

In the paper's experiments, MCTS found a near-optimal passing order in much less time than exhaustive search. It used UCB1 method to choose which branch of the tree to explore next. It also used heuristic rules to complete the remaining passing order during rollouts.

To use this approach in my simulator, I had to adapt it to the intersection layout and how vehicles moved. I divided the intersection into four conflict subzones and added replan() to update schedules of vehicles that had not yet entered. 
After getting the heuristic + MCTS approach, I ran into a problem: the paper's assumption that vehicles cross the conflict zone at a constant speed v0 didn't hold in the real world. 
A vehicle starting from a stop accelerates gradually instead of instantly reaching v0, so the schedule was underestimating how long it actually occupied a subzone. I fixed this by predicting each vehicle's actual entry speed and tracking both its arrival and exit time per subzone, instead of assuming constant speed throughout

That left me with three schedulers. I made the FIFO baseline reuse the exact same subzone model as the MCTS one, so the only difference between them is how the passing order gets picked. Otherwise I'd be comparing two different models of the intersection, not two algorithms.

| Algorithm        | What it does                                                                                                   | Low load (8 veh/min) | High load (24 veh/min) | Violations |
| ---------------- | -------------------------------------------------------------------------------------------------------------- | -------------------- | ---------------------- | ---------- |
| `algo_fcfs`      | FIFO on the paper's model. Earliest ETA goes first, no overtaking within a lane, replanned every 2 s.           | 1.4–2.0 s, 15–16 veh/min | 8.1 s, 16 veh/min      | 0          |
| `algo_mcts_heur` | Xu et al.'s MCTS with the heuristic rollout, same model.                                                        | 1.7 s, 14–15 veh/min | **6.8 s, 26 veh/min**  | 0          |
| `algo_mcts_ad`   | `algo_mcts_heur` plus my fix: real entry-speed prediction per vehicle, and a gate at the stop line that turns a missed slot into a wait instead of a conflict. | 3.8 s, 10 veh/min    | 10.6 s, 16 veh/min     | 0          |

At low load the three are basically tied, which is what the paper says should happen: with few cars there's no order worth optimizing. At high load the search pays off. MCTS gets about 60% more vehicles through per minute than FIFO, with lower delay.

One thing I got wrong on the way: my first baseline used a single conflict box per axis and estimated arrival as `distance / v0`, ignoring the car's current speed. It looked better than MCTS at low load, but only because its lane gap (1.1 s) was tighter than the paper's Δ (1.5 s). And at high load it caused real safety violations, because a stopped car was promised a slot it physically couldn't reach. Numbers above are single 45 s runs per cell, so I'd trust the ranking more than the exact values.

### 5. Result

Each one has a clear trade-off:

| Algorithm        | Pros                                                                                                   | Cons                                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| `algo_fcfs`      | Zero search cost, fully predictable order, matches MCTS when traffic is light.                         | Never reorders across directions, so once conflicts pile up it just queues them. Throughput caps early.        |
| `algo_mcts_heur` | Best delay and throughput under load. Reorders vehicles across directions to fill gaps FIFO leaves.    | Assumes every car crosses at v0. Its slot promises are optimistic for a car restarting from a stop, and nothing checks whether a car actually made its slot. |
| `algo_mcts_ad`   | Only one whose slot times match what the car will physically do. Safety is enforced at the stop line, not by hoping the prediction was right. | Slowest in every condition I measured. Its extra margins cost throughput, and the case it fixes barely shows up at this simulator's acceleration. |

My own extension is the slowest one in every condition I tried. That surprised me, but it makes sense: the situation it was built for, cars restarting slowly from a stop, turns out to be mild at this simulator's acceleration limit, so there wasn't much for the more honest model to correct. Meanwhile the extra safety margins it adds cost throughput.

Still, I think it's the more realistic and the more stable of the three, for two reasons.

It's more realistic because it's the only one that models both halves of the trip with actual vehicle physics. `algo_mcts_heur` already predicts *when* a car reaches the intersection using its current speed and acceleration limit, but once the car is inside it assumes constant v0, so a car that's still accelerating gets the same crossing time as one that never slowed down. `algo_mcts_ad` predicts the entry speed and computes the crossing time from that, so a car that stopped gets a longer slot and a car that never stopped gets a shorter one. Its `t1` is what the car will actually do, not what the paper assumed.

It's more stable because it doesn't rely on that prediction being perfect. With `algo_mcts_heur`, if a car arrives late, it just enters late, and whether that overlaps with someone else is left to chance and the next replan. With `algo_mcts_ad`, a car that misses its window by more than 0.5 s is held at the line and gets a new slot on the next replan. The failure mode changes from "possible conflict" to "wait a bit," and the missed-slot counter tells me how often the prediction was off. It also freezes slots that are about to start, so a stopped car's slot can't keep sliding into the future every time the plan is recomputed, which is a loop I actually hit before adding that. None of this shows up as better numbers in the table above. It shows up as the algorithm not breaking when conditions get worse, and I haven't run the stress test (lower acceleration, longer crossing, communication delay) that would demonstrate that yet.

Before settling on these three, I went through several other schedulers (cost-benefit yielding, emergency priority, P2P visibility). They're archived now, but two things I learned from them shaped the final comparison.

First, the yielding rewrite flipped the result completely, same idea, opposite outcome, because the second version measures the actual delay instead of guessing at a proxy for it:

| Algorithm                 | Delay vs. FCFS | Note                          |
| -------------------------- | -------------- | ------------------------------ |
| Threshold rule (v1)         | −59% (worse)   | Nearly every request yielded  |
| Cost-benefit rewrite (v2)   | +28–31%        | Same idea, measures the outcome |

Second, the subzone fix restored the parallel-crossing case the MCTS paper's advantage actually depends on. Without it I would have concluded the paper's algorithm just doesn't help here, which wasn't true. I'd just implemented a version of the intersection where its advantage couldn't exist.

<div class="reflection" markdown="1">

### 6. Reflection

I realized that a benchmark result depends as much on the protocol and setup around an algorithm as on the algorithm itself. Both times MCTS looked unimpressive, the algorithm wasn't actually the problem, my implementation of the surrounding conditions was. I also learned to benchmark a "smarter" heuristic before trusting it, since my first cost-benefit rule made things worse despite sounding reasonable.

</div>
