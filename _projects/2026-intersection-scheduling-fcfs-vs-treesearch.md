---
title: "Cooperative Intersection Scheduling: FCFS, Tree Search, and GNN"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I implemented FCFS, paper-based MCTS, a safety extension, and a GNN that learns which vehicle to send next. In a five-seed high-load comparison, the GNN reduced mean vehicle delay by 12.3% and increased throughput by 5.4% relative to FCFS."
thumbnail: /assets/img/projects/v2v-intersection-gnn.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/v2v-intersection-coordinator
---

Two self-driving cars can each know how to drive and still need to agree on who enters an intersection first. I wanted to see how that decision could work, so I built a browser simulation of an intersection without traffic lights.

I began with First Come, First Served (FCFS), implemented a paper-based Monte Carlo Tree Search (MCTS) scheduler, and added rules for cars that could not follow their assigned crossing time. Later, I trained a graph neural network (GNN) to choose the next vehicle from the queue. The four implementations let me inspect both the passing order and what happens when cars try to follow it.

## The intersection I modeled
{: #study-problem }

Cars approach from four directions and request a crossing window. The intersection has four conflict areas, so the scheduler can check whether two movements would use the same space at the same time. An IDM car-following model controls acceleration and following distance.

### A shared setup
{: #study-scope }

The arrival attempts are randomized and jittered. They are Poisson-like, rather than an exact Poisson process. This is a simulation of the coordination decision. I did not measure how reliably a real vehicle-to-vehicle network would deliver messages.

My first FCFS implementation used one conflict area per axis, estimated arrival as `distance / v0` without accounting for current speed, and used a shorter lane gap than the paper-based model. That made the early comparison misleading and produced violations under higher demand. Before the final comparisons, I made the schedulers use the same four-area intersection model.

## From arrival order to a crossing plan
{: #study-process }

### FCFS and tree search

With FCFS, the car expected to arrive first gets the first opportunity to cross. Cars in the same lane cannot overtake each other in the schedule. The scheduler still reserves space and time, but it does not try alternative passing orders. This can leave gaps that a different order could use.

For MCTS, I implemented the approach described by Xu et al. The search tries different next vehicles and evaluates the resulting plans for total delay. UCB1 guides which branch to explore. Heuristic rules complete the remaining order during rollouts, giving the search a plan to evaluate.

There are two different motion assumptions in this implementation. The earliest-arrival estimate lets a car accelerate at a fixed maximum rate until it reaches the speed limit. The crossing schedule then assumes constant-speed travel through the intersection. That distinction mattered once I looked at cars entering slowly.

### What happens when a car misses its slot?

A stopped car cannot instantly reach its crossing speed. I extended MCTS to estimate entry speed from the car’s current speed and remaining distance, then account for acceleration when estimating entry and exit times for each conflict area. It is still a simplified motion prediction.

I also added a stop-line gate. If a car misses its assigned window by more than 0.5 seconds, it waits for a new schedule instead of entering late. That gives a defined response to a missed slot. Whether the extra waiting prevents violations needs its own test.

The implementations are `algo_fcfs`, `algo_mcts_heur`, and `algo_mcts_ad`. FCFS replans every two seconds. My extension changes both entry-speed prediction and missed-slot handling, so the comparison does not separate their effects.

## Teaching a GNN to choose the next vehicle

I adapted the relational graph model described by Klimke et al. (ITSC 2022) to this reservation problem. This is my implementation, rather than the authors’ code or a pretrained policy.

Each waiting vehicle is a node. Edges represent same-lane precedence and shared conflict areas, not a measured wireless network. The model encodes 10 vehicle features into 64 dimensions and two edge features into 32 dimensions. Two relational message-passing layers pass information between relevant neighbors, and a decoder produces one score per vehicle. The number of nodes changes with the queue. The value 64 is the representation width.

The first version predicted candidate delay values with Smooth L1 loss. I changed the target to match the choice the scheduler actually makes: which eligible vehicle should go next?

- In `train_gnn.py`, I enumerate valid continuations for generated scenes with 2–6 vehicles and find the candidates that can achieve minimum total schedule delay. Candidates tied within 10⁻⁶ seconds all count as correct.
- The loss is the negative log of the combined selection probability assigned to those optimal candidates. Ineligible vehicles and padded entries are excluded.
- In `gnn_graph.py`, inference uses masked `argmax` of selection scores instead of masked `argmin` of predicted delays. The graph is rebuilt after each choice to reflect the updated reservations. Same-lane eligibility prevents a following car from being scheduled ahead of its predecessor.
- The network in `gnn_model.py` stayed the same, but its output now means a selection score, not seconds. Version and objective checks reject the old regression checkpoint. The scheduler in `algo_gnn.py` loads `models/gnn_selection.pt`.

I retrained on 1,200 generated samples for 40 epochs, keeping the data-generation method and model widths. Training and static evaluation use separate random seeds. At runtime, the GNN chooses the order and the existing timing evaluator assigns crossing reservations. The simulator’s movement model stays the same.

The comparison below tests this selection GNN against FCFS. It does not isolate what changed because of the new learning objective. That would require a matched comparison with the old regression GNN.

## What the short FCFS and MCTS runs showed
{: #study-results }

The original runs lasted 45 seconds each. I used arrival settings of 8 and 24 vehicles/minute **per approach**, with four approaches. The table records delay and throughput from those short runs, not steady-state capacities.

| Scheduler | Delay at low load (s) | Throughput at low load (veh/min) | Delay at high load (s) | Throughput at high load (veh/min) |
| --- | ---: | ---: | ---: | ---: |
| FCFS | 1.4–2.0 | 15–16 | 8.1 | 16 |
| MCTS with heuristic rules | 1.7 | 14–15 | 6.8 | 26 |
| MCTS with entry-speed prediction and stop-line gate | 3.8 | 10 | 10.6 | 16 |

At light load, there was little difference between FCFS and MCTS. With busier traffic, MCTS found an order with less delay and more completed crossings. My extension added waiting at both loads.

The extension’s minimum measured gap under load was 2.9 m, compared with 2.0 m for the comparison. All three final versions recorded zero violations in these runs. The larger gap does not establish a safety benefit on its own. These measurements concern vehicle delay and throughput, not the computation time needed to choose an order.

Earlier experiments also made me wary of calling a rule helpful before checking its effect. An initial threshold-based yielding rule increased delay by 59% relative to FCFS. After I replaced its proxy with actual delay, the reported result became a 28–31% improvement. Those were separate experiments. I did not run an MCTS-without-heuristics comparison, so these numbers cannot identify the contribution of the rollout heuristics.

## A longer comparison of GNN and FCFS

For this comparison, each run had a 60-second warm-up followed by a 300-second measurement window. I used the same two arrival settings and seeds 101, 202, 303, 404, and 505 for each scheduler, making 20 runs in total.

Paired runs had identical attempted arrivals, checked by matching arrival-pattern hashes. FCFS and GNN shared the crossing timing evaluator and two-second replanning interval. Neither used the stop-line gate.

| Arrival setting per approach | Scheduler | Mean delay (s) | Mean completions / 300 s | Mean throughput (veh/min) | Violations across 5 runs |
| --- | --- | ---: | ---: | ---: | ---: |
| 8 veh/min | FCFS | 1.701 | 156.6 | 31.32 | 0 |
| 8 veh/min | GNN | 1.523 | 156.6 | 31.32 | 0 |
| 24 veh/min | FCFS | 75.264 | 144.2 | 28.84 | 1 |
| 24 veh/min | GNN | 66.033 | 152.0 | 30.40 | 0 |

At light load, the GNN reduced mean delay by 10.5%, with unchanged throughput. At high load, mean delay fell by 12.3% and throughput rose by 5.4%. These are averages across five seeds. The GNN did not necessarily improve every individual run.

Here, delay means extra whole-route travel time above free-flow travel time for vehicles that completed their route during the measurement window. That includes vehicles spawned during warm-up and excludes those still waiting at the end. It is not wireless latency. Throughput counts completions over the full 300 seconds, rather than using the simulator’s rolling last-minute display.

The simulator drops a spawn attempt when an entrance is blocked. Matching attempted arrivals therefore does not guarantee matching admitted traffic. The experiment also leaves unfinished vehicles out of the delay measure. Those details matter when interpreting a heavily loaded intersection.

The outputs are saved in `results/gnn_selection_v2/runs.json` and `summary.json`. The paired comparison runs through `run_gnn_comparison.py`.

### Recordings from the simulator

These clips show how the schedulers run. Their on-screen telemetry is separate from the benchmark results.

<details>
<summary>Watch the GNN choose vehicles</summary>
<figure>
<img src="{{ '/assets/img/projects/v2v-intersection-gnn.gif' | relative_url }}" alt="GNN intersection scheduling with the GNN button selected and an arrival rate of 24 vehicles per minute per approach" loading="lazy">
<figcaption>Recorded October 3, 2026, with seed 20261003. The 12-second clip follows a 60-second warm-up. Arrival rate is 24 vehicles/minute per approach, communication range is 55 m, and speed limit is 14 m/s. The GNN chooses the order and the shared timing model assigns reservations.</figcaption>
</figure>
</details>

<details>
<summary>Watch the earlier MCTS run</summary>
<figure>
<img src="{{ '/assets/img/projects/v2v-intersection-scheduling.gif' | relative_url }}" alt="Cars approaching a simulated intersection while MCTS with heuristic rules assigns crossing times" loading="lazy">
<figcaption>The earlier MCTS demonstration uses an arrival setting of 22 vehicles/minute. The high-load measurements use 24.</figcaption>
</figure>
</details>

## What I can conclude from these tests
{: #study-limitations }

The GNN did better than FCFS on average in the repeated comparison. The original MCTS runs suggest that changing the passing order can help under load. Their durations and measurement methods differ, though, so I cannot combine them into a ranking of all four approaches. One short run per setting for MCTS and five seeds for the GNN also leave uncertainty about other traffic patterns and statistical significance.

None of these runs establishes real-world safety or reliable communication. Delayed and lost messages were not measured. Neither 45-second runs nor 300-second windows establish long-run capacity, and the GNN comparison did not measure deadline misses or fairness. Its training scenes were small and synthetic.

The part I keep coming back to is the gap between a plan and a car following it. My extension addressed that gap, but it also increased waiting, without demonstrating fewer violations. The GNN improved the average result against FCFS, but I still need the old-versus-new comparison to understand how much the objective change contributed.

## The comparisons still missing
{: #study-next }

I would first put all four schedulers through one protocol, with matched arrivals and longer measurement windows. An external queue would retain vehicles whose spawn attempts are currently dropped.

For the GNN, I would compare regression and selection objectives under matched training conditions and driving seeds, then try larger queues and held-out traffic patterns. For the motion rules, slower acceleration, longer crossing times, and delayed information would make it harder for cars to follow their slots. Changing entry-speed prediction and the stop-line gate one at a time would show which part causes the extra waiting. Different MCTS rollout rules need the same kind of comparison.
