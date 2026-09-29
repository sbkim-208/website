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

**At low load, FIFO and MCTS tie.** At 8 veh/min they land within noise of each other (1.4–2.0 s vs. 1.7 s). With few cars there is no order worth optimizing, and FIFO gets that result for free: no search, and a fully predictable order.

**At high load, MCTS wins.** At 24 veh/min it moves 26 veh/min through with 6.8 s delay, against FIFO's 16 veh/min and 8.1 s. It reorders cars across directions to fill gaps that FIFO leaves empty.

**My extension is the slowest everywhere, but not for the reason I expected.** The case it was built for, cars restarting from a stop, turns out to be mild at this simulator's acceleration limit, so there was little for the more honest model to correct, while its extra margins cost throughput. All three had zero violations, so I can't claim it's safer from the numbers. What it does have is a larger margin (minimum gap 2.9 m vs. 2.0 m under load) and a failure mode that doesn't depend on the prediction being right: a car that misses its window by more than 0.5 s is held at the stop line and rescheduled instead of entering late. Whether that turns into fewer violations under harder conditions (lower acceleration, longer crossing, communication delay) is the stress test I still need to run.

| Algorithm        | Pros                                                                                   | Cons                                                                                                  |
| ---------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `algo_fcfs`      | Zero search cost, fully predictable, matches MCTS when traffic is light.               | Never reorders across directions, so throughput caps early once conflicts pile up.                    |
| `algo_mcts_heur` | Best delay and throughput under load.                                                  | Assumes every car crosses at v0, and nothing checks whether a car actually made its slot.             |
| `algo_mcts_ad`   | Slot times match what the car will physically do. Safety is enforced at the stop line. | Slowest in every condition measured. The case it fixes barely shows up at this simulator's acceleration. |

Before settling on these three, I went through several other schedulers (cost-benefit yielding, emergency priority, P2P visibility). Two lessons from them shaped the final comparison. A threshold-based yielding rule made delay 59% worse than FCFS because nearly every request yielded; rewriting it to measure the actual delay instead of a proxy turned the same idea into a 28–31% improvement. And the subzone fix restored the parallel-crossing case the MCTS paper's advantage depends on. Without it I would have concluded the algorithm just doesn't help here, when really I had built a version of the intersection where its advantage couldn't exist.

<div class="reflection" markdown="1">

### 6. Reflection

I realized that a benchmark result depends as much on the protocol and setup around an algorithm as on the algorithm itself. Both times MCTS looked unimpressive, the algorithm wasn't actually the problem, my implementation of the surrounding conditions was. I also learned to benchmark a "smarter" heuristic before trusting it, since my first cost-benefit rule made things worse despite sounding reasonable.

</div>
