---
title: "Cooperative Intersection Scheduling: FCFS, Tree Search, and GNN"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I implemented FCFS, paper-based MCTS, a safety extension, and a GNN that learns which vehicle to send next. In a five-seed high-load comparison, the GNN reduced mean vehicle delay by 12.3% and increased throughput by 5.4% relative to FCFS."
thumbnail: /assets/img/projects/v2v-intersection-scheduling.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/v2v-intersection-coordinator
---

{% include project-study-style.html %}

<div id="project-study">
<header class="study-hero">
<span class="study-eyebrow">Connected vehicles · tree search · learned scheduling</span>
<h2>How would self-driving cars decide who goes first?</h2>
<span class="study-badge">Four approaches implemented</span> <span class="study-badge amber">Simulation study</span>
<p>If connected autonomous vehicles become part of everyday traffic, they will need to do more than drive themselves. They will also need to exchange information and coordinate their movements. I started this project because I wanted to understand how that coordination could work.</p>
<p>I focused on one situation: an intersection without traffic lights. I first implemented a simple first-come-first-served rule, then a paper-based method that searches for a better passing order. I then added rules for vehicles that could not follow their scheduled crossing time, and implemented a graph neural network (GNN) to choose the next vehicle from the current queue.</p>
<p><strong>The project now includes four approaches.</strong> The original short runs compared FCFS, MCTS, and my safety extension. A separate, longer comparison tested the GNN against FCFS using five paired arrival seeds. These experiments helped me distinguish choosing an efficient order, learning that choice, and checking whether vehicles can follow the plan.</p>
</header>
<div class="study-metrics" aria-label="GNN comparison results at a glance">
<div class="study-metric"><span class="study-eyebrow">GNN at high load</span><strong>12.3%</strong><small>Lower mean vehicle delay<br>66.03 s vs. FCFS: 75.26 s</small></div>
<div class="study-metric"><span class="study-eyebrow">Throughput gain</span><strong>5.4%</strong><small>30.40 vehicles/min<br>FCFS: 28.84 vehicles/min</small></div>
<div class="study-metric"><span class="study-eyebrow">Paired evaluation</span><strong>5 seeds</strong><small>60 s warm-up + 300 s measurement<br>At each arrival setting</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-scope">Study setup</a><a href="#study-process">Four approaches</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">One Intersection, One Shared Decision</h2>
<p>Imagine two self-driving cars approaching the same intersection from different directions. Each car may be able to drive on its own, but they still need to agree on who enters first. If their paths overlap, planning both crossings independently could give them incompatible instructions.</p>
<p>I wanted to understand how information about an approaching vehicle turns into a crossing plan. To keep the problem manageable, I built a browser simulation in which a vehicle requests a time window and a scheduling algorithm decides when it can enter.</p>
<h3 id="study-scope">What I modeled</h3>
<p>Cars approach from four directions. The intersection is divided into four smaller areas, so the scheduler can check which movements would occupy the same area at the same time. The cars also follow a movement model that controls how they accelerate and follow the car ahead.</p>
<p>This models the coordination decision. It does not establish that a real vehicle-to-vehicle communication network would deliver every message reliably or on time.</p>
<figure><img src="{{ '/assets/img/projects/v2v-intersection-scheduling.gif' | relative_url }}" alt="Cars approaching a simulated intersection while the MCTS scheduler chooses crossing times" loading="lazy"><figcaption>The browser simulator using MCTS with heuristic rules. This demonstration uses an arrival setting of 22 vehicles/minute; the high-load measurements below use 24.</figcaption></figure>
</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">What I Wanted to Understand</h2>
<ul>
<li><strong>Coordination:</strong> how does a vehicle’s request become a shared passing order?</li>
<li><strong>Search:</strong> how does the paper’s MCTS method differ from simply serving vehicles in arrival order, and what do its heuristic rules contribute?</li>
<li><strong>Learning:</strong> can a graph model learn which eligible vehicle should go next to reduce total schedule delay?</li>
<li><strong>Safety rules:</strong> what should happen when a car cannot reach the intersection at its assigned time?</li>
</ul>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">From a Simple Queue to Search and Learned Choices</h2>
<h3>1. FCFS: start with a predictable queue</h3>
<p>I first implemented <strong>First Come, First Served (FCFS)</strong>. Think of a queue: the car expected to arrive first gets the first opportunity to cross. Cars in the same lane cannot overtake each other in the schedule.</p>
<p>This gave me a clear starting point for understanding coordination. The scheduler still needs to reserve the areas each car will use, but it does not search through different passing orders. Its weakness is that arrival order can leave gaps that another order might use.</p>
<h3>2. MCTS with heuristic rules: try different passing orders</h3>
<p>I then implemented an approach from Xu et al.’s paper. <strong>Monte Carlo Tree Search (MCTS)</strong> explores possible choices about which car should go next, evaluates the resulting plans, and uses those results to guide further search. Instead of accepting the queue as it is, it looks for an order with less total delay.</p>
<p>The <strong>heuristic rules</strong> are practical shortcuts used during that search. After the search chooses part of an order, these rules help complete the remaining order so the plan can be evaluated. Without considering their role, I would be treating MCTS as a name rather than understanding how this implementation produces a useful plan.</p>
<p>The MCTS scheduler already accounts for acceleration when estimating the earliest arrival: it assumes the car accelerates at a fixed maximum rate until it reaches the speed limit. But when assigning the crossing schedule, it treats movement through the intersection as constant-speed travel. These are two different parts of the calculation.</p>
<p>For the final comparison, I made FCFS and MCTS use the same model of the intersection’s four areas. That let me compare how they chose the passing order without also changing the layout they were planning around.</p>
<p class="study-callout"><strong>Searching for a good order and executing it are separate problems.</strong> A plan can look efficient while depending on a car reaching its assigned place at an unrealistic time.</p>
<h3>3. My MCTS extension: check whether the car can follow the plan</h3>
<p>A stopped car cannot instantly reach its crossing speed. The second version already considers acceleration for arrival-time estimates, but its crossing schedule assumes the car travels through the intersection at the target speed. A car entering slowly can take longer to clear it. I extended the MCTS version to estimate entry speed from its current speed and remaining distance, then include acceleration when estimating when it would enter and leave each area. This remains a simplified prediction, not a guarantee of the car’s actual motion.</p>
<p>I also added a rule at the stop line: <strong>if a car misses its assigned window by more than 0.5 seconds, it waits for a new schedule instead of entering late.</strong> The aim was to make the response to a missed time slot more cautious. Whether that produces fewer safety violations needs separate testing.</p>
<h3>4. GNN: learn which vehicle should go next</h3>
<p>I adapted the relational graph model described by Klimke et al. (ITSC 2022) to the intersection reservation problem. This is my implementation and adaptation, not the authors’ code or a pretrained policy. Each waiting vehicle is a node. Edges represent same-lane precedence and shared intersection conflict areas; they do not represent a measured wireless communication network.</p>
<p>The model encodes 10 vehicle features into a 64-dimensional representation and two edge features into 32 dimensions. Two relational message-passing layers let each vehicle use information from relevant neighbors. A decoder produces one score per vehicle. The node count changes with the queue size; 64 is the representation width, not the number of vehicles.</p>
<h4>What I changed in the code</h4>
<ul>
<li><strong><code>train_gnn.py</code> — change the target.</strong> Previously, the model regressed candidate delay values with Smooth L1 loss. Now, for small generated scenes with 2–6 vehicles, I enumerate valid continuations and identify which next candidates can achieve the minimum total schedule delay. All candidates tied within 10<sup>−6</sup> seconds are accepted as correct.</li>
<li><strong><code>train_gnn.py</code> — change the loss.</strong> The new loss is the negative log of the combined selection probability of the optimal candidates. It rewards choosing any optimal candidate, while excluding ineligible vehicles and padded entries.</li>
<li><strong><code>gnn_graph.py</code> — change the decision.</strong> Inference changed from masked <code>argmin</code> of predicted delay to masked <code>argmax</code> of selection scores. After each choice, the graph is rebuilt to reflect the updated reservation state. Same-lane eligibility prevents scheduling a following vehicle ahead of its predecessor.</li>
<li><strong><code>gnn_model.py</code> and <code>algo_gnn.py</code> — distinguish the new weights.</strong> The network structure stayed the same, but its output now means a selection score rather than seconds. Version and objective checks reject the old regression checkpoint, and the scheduler loads <code>models/gnn_selection.pt</code>.</li>
</ul>
<p>I retrained the same two-layer model on 1,200 generated samples for 40 epochs, retaining the data-generation method and model widths. Training and static evaluation use separate random seeds. At runtime, the GNN selects the order; the existing timing evaluator still assigns crossing reservations. I did not replace the simulator’s movement model.</p>
<p class="study-callout"><strong>The learning task now matches the scheduling decision.</strong> Instead of asking “How many seconds does each choice cost?”, the model learns “Which eligible vehicle should go next?” The new model’s comparison with FCFS does not isolate the effect of this objective change from the rest of training.</p>
<div class="study-table" role="region" aria-label="Four implemented approaches" tabindex="0">
<table><thead><tr><th scope="col">Approach</th><th scope="col">Main idea</th><th scope="col">Strength</th><th scope="col">Trade-off</th></tr></thead><tbody>
<tr><td>1. FCFS</td><td>Follow expected arrival order.</td><td>Simple and predictable; no search over alternative orders.</td><td>Does not rearrange cars from different directions to reduce waiting.</td></tr>
<tr><td>2. MCTS + heuristic rules</td><td>Search alternative orders; use practical rules to complete trial plans.</td><td>Found a lower-delay order in the high-load run.</td><td>Arrival estimates include acceleration; the assigned crossing schedule assumes a fixed crossing speed.</td></tr>
<tr><td>3. MCTS + entry-speed prediction and stop-line gate</td><td>Keep the search, but account for acceleration and reschedule missed slots.</td><td>Adds a defined response when a vehicle cannot follow its time window.</td><td>Added waiting and reduced throughput in the reported runs; a safety benefit is not yet established.</td></tr>
<tr><td>4. GNN candidate selection</td><td>Learn the next eligible vehicle from graph-structured traffic state.</td><td>Lower mean delay than FCFS in the five-seed comparison; higher throughput at high load.</td><td>Trained on small synthetic scenes; results do not establish generalization to all traffic conditions.</td></tr>
</tbody></table></div>
<h3>How a request becomes a crossing</h3>
<ol class="study-flow"><li><span>01 · REQUEST</span><strong>Ask to cross</strong>An approaching car requests a time window.</li><li><span>02 · ORDER</span><strong>Choose who goes next</strong>FCFS follows the queue; MCTS searches alternatives; GNN scores eligible candidates.</li><li><span>03 · PLAN</span><strong>Reserve space and time</strong>Estimate when each part of the intersection will be occupied.</li><li><span>04 · ACT</span><strong>Cross or wait</strong>My extension holds cars that miss their window and schedules them again.</li></ol>
<details><summary>Technical details and an early implementation mistake</summary>
<p>The implemented schedulers are <code>algo_fcfs</code>, <code>algo_mcts_heur</code>, <code>algo_mcts_ad</code>, and <code>algo_gnn</code>. The MCTS implementation uses UCB1 to guide which search branch to explore and heuristic rules to complete rollout orders. Vehicles follow an IDM car-following model with randomized, jittered arrival attempts. The arrival process is Poisson-like rather than an exact Poisson process.</p>
<p>My first FCFS version used one conflict area per axis and estimated arrival as <code>distance / v0</code>, without accounting for current speed. It also used a shorter lane gap than the paper-based model. That made the early comparison misleading and produced violations under higher demand. I changed the shared intersection model before comparing the final versions.</p>
</details>
</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">What Happened When I Compared Them</h2>
<p>The original FCFS/MCTS/safety-extension tests below lasted 45 seconds each. I later ran a separate GNN-versus-FCFS evaluation with a 60-second warm-up and a 300-second measurement window per run. Both experiments use arrival settings of 8 and 24 vehicles/minute per approach, with four approaches. Their durations and measurement methods differ, so their numbers should not be combined into a four-way ranking.</p>
<h3>Original short-run comparison: FCFS, MCTS, and the safety extension</h3>
<h3>With light traffic, searching made little difference</h3>
<p>FCFS reported 1.4–2.0 seconds of delay, compared with 1.7 seconds for MCTS with heuristic rules. When there are few competing cars, changing the order may offer little benefit. My extension reported 3.8 seconds.</p>
<h3>With busier traffic, MCTS with heuristic rules performed best</h3>
<p>The MCTS version reported 6.8 seconds of delay, compared with FCFS’s 8.1 seconds. It also moved more cars through the intersection during the measured run. Here, exploring a different order helped.</p>
<figure class="study-chart"><div class="study-chart-body"><h3>Delay at the high-load setting</h3><p class="study-chart-unit">Delay in seconds · lower is better</p><div class="study-chart-row"><span>FCFS</span><div class="study-chart-track"><span style="width:76.415%"></span></div><strong>8.1</strong></div><div class="study-chart-row"><span>MCTS + rules</span><div class="study-chart-track"><span style="width:64.151%"></span></div><strong>6.8</strong></div><div class="study-chart-row"><span>My MCTS extension</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>10.6</strong></div></div><figcaption>Figure 1. Reported delay at an arrival setting of 24 vehicles/min. Each scheduler was measured in a single 45-second run.</figcaption></figure>
<figure class="study-chart"><div class="study-chart-body"><h3>How many cars passed through?</h3><p class="study-chart-unit">Vehicles per minute · higher is better</p><div class="study-chart-row"><span>FCFS</span><div class="study-chart-track"><span style="width:61.538%"></span></div><strong>16</strong></div><div class="study-chart-row"><span>MCTS + rules</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>26</strong></div><div class="study-chart-row"><span>My MCTS extension</span><div class="study-chart-track"><span style="width:61.538%"></span></div><strong>16</strong></div></div><figcaption>Figure 2. Short-run throughput at the same high-load setting; these measurements are not steady-state capacities.</figcaption></figure>
<details><summary>View exact low- and high-load measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Algorithm</th><th scope="col">What it does</th><th scope="col">Low load (8 veh/min)</th><th scope="col">High load (24 veh/min)</th><th scope="col">Violations</th></tr></thead><tbody><tr><td><code>algo_fcfs</code></td><td>FCFS on the shared intersection model. Earliest ETA goes first, no overtaking within a lane, replanned every 2 s.</td><td>1.4–2.0 s, 15–16 veh/min</td><td>8.1 s, 16 veh/min</td><td>0</td></tr><tr><td><code>algo_mcts_heur</code></td><td>Xu et al.'s MCTS with the heuristic rollout, same model.</td><td>1.7 s, 14–15 veh/min</td><td><strong>6.8 s, 26 veh/min</strong></td><td>0</td></tr><tr><td><code>algo_mcts_ad</code></td><td><code>algo_mcts_heur</code> plus my fix: real entry-speed prediction per vehicle, and a gate at the stop line that turns a missed slot into a wait instead of a conflict.</td><td>3.8 s, 10 veh/min</td><td>10.6 s, 16 veh/min</td><td>0</td></tr></tbody></table></div></details>
<h3>My extension added caution, but it also added waiting</h3>
<p>The extension reported 10.6 seconds of delay and 16 vehicles/minute in the high-load run. It was slower than the other two approaches. Accounting for entry speed and holding missed slots did not produce a faster crossing plan in these conditions.</p>
<p>Its minimum measured gap was larger: 2.9 m compared with 2.0 m for the comparison under load. However, <strong>all three final versions recorded zero violations in these runs</strong>. A larger gap and an extra rule do not, by themselves, prove that my version is safer.</p>
<p class="study-callout"><strong>“Uses MCTS” does not mean “always gives the shortest wait.”</strong> The result depends on traffic conditions, the rules used to build a plan, and how the cars actually move. These tests compare vehicle delay and cars passing through—not the time the computer spends calculating an answer.</p>
<details><summary>What the experiments say about heuristic rules</summary>
<p>In earlier scheduler experiments, an initial threshold-based yielding rule increased delay by 59% relative to FCFS. After I changed that rule to use actual delay instead of a proxy, the reported result became a 28–31% improvement. These were separate experiments, not the three-way comparison above and not an isolated test of the MCTS rollout rules.</p>
<p>That experience reinforced why the details of a rule matter. This page does not include an MCTS-without-heuristics comparison, so I cannot assign a separate performance gain to the rollout heuristics.</p>
</details>
<h3>New five-seed comparison: GNN versus FCFS</h3>
<p>I ran both schedulers at each load using seeds 101, 202, 303, 404, and 505, for 20 runs in total. Each paired run used identical arrival attempts, verified by matching arrival-pattern hashes. FCFS and GNN used the same crossing timing evaluator and a two-second replanning interval, without the stop-line gate.</p>
<div class="study-table" role="region" aria-label="Five-seed GNN versus FCFS results" tabindex="0">
<table><thead><tr><th scope="col">Arrival setting per approach</th><th scope="col">Scheduler</th><th scope="col">Mean delay (s)</th><th scope="col">Mean completed vehicles / 300 s</th><th scope="col">Mean throughput (veh/min)</th><th scope="col">Recorded violations across 5 runs</th></tr></thead><tbody>
<tr><td>8 veh/min</td><td>FCFS</td><td>1.701</td><td>156.6</td><td>31.32</td><td>0</td></tr>
<tr><td>8 veh/min</td><td>GNN</td><td>1.523</td><td>156.6</td><td>31.32</td><td>0</td></tr>
<tr><td>24 veh/min</td><td>FCFS</td><td>75.264</td><td>144.2</td><td>28.84</td><td>1</td></tr>
<tr><td>24 veh/min</td><td>GNN</td><td>66.033</td><td>152.0</td><td>30.40</td><td>0</td></tr>
</tbody></table></div>
<p>At light load, GNN reduced mean delay by <strong>10.5%</strong>, with unchanged mean throughput. At high load, it reduced mean delay by <strong>12.3%</strong> and increased mean throughput by <strong>5.4%</strong>. These are averages across five seeds, not improvements in every individual run.</p>
<p>Delay means extra whole-route travel time above free-flow travel time for vehicles completing their route during the measurement window, including vehicles spawned during warm-up. It is not wireless latency and excludes vehicles still waiting at the end. Throughput counts completions over the full 300-second window, rather than the simulator’s rolling last-minute display.</p>
<p>The simulator drops a spawn attempt when its entrance is blocked. Identical attempted arrivals therefore do not guarantee identical admitted traffic. This experiment also did not compare the old regression GNN and the new selection GNN under matched driving conditions, so it cannot establish that changing the objective alone caused the observed advantage over FCFS.</p>
<p>Reproducible outputs are saved in <code>results/gnn_selection_v2/runs.json</code> and <code>summary.json</code>; <code>run_gnn_comparison.py</code> runs the paired comparison.</p>
</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">What These Tests Do Not Establish</h2>
<ul>
<li><strong>A general winner.</strong> The original tree-search comparison used one short run per setting. The GNN comparison used five seeds, but does not establish statistical significance, superiority in every run, or a general winner among all four approaches.</li>
<li><strong>Real-world safety.</strong> Zero recorded violations in a simulation is not a guarantee for autonomous vehicles on real roads.</li>
<li><strong>Reliable communication.</strong> These results do not measure delayed or lost messages between vehicles.</li>
<li><strong>The isolated value of each rule.</strong> The extension changes both speed prediction and missed-slot handling. The comparison does not separate their effects, or isolate the benefit of MCTS’s heuristic rules.</li>
<li><strong>Long-run capacity and admitted demand.</strong> Neither the original 45-second runs nor the new 300-second windows establish steady-state capacity. Blocked entrances drop spawn attempts, and completion-only delay excludes unfinished vehicles.</li>
<li><strong>Learning-objective attribution.</strong> A matched regression-GNN versus selection-GNN experiment is still needed to isolate the objective change. Deadline miss rate and fairness were not measured in the new comparison.</li>
</ul>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">What I Would Test Next</h2>
<p class="study-callout amber">I now have four implementations and a repeated GNN-versus-FCFS comparison. The next step is to test how broadly the learned choices generalize and when the safety rules justify their extra waiting.</p>
<ol class="study-roadmap">
<li><strong>Compare all four on one protocol.</strong> Use matched arrivals and longer measurement windows, with an external queue to retain blocked arrivals.</li>
<li><strong>Isolate the learning objective.</strong> Compare the regression and selection GNNs with matched training conditions and driving seeds; include larger queues and held-out traffic patterns.</li>
<li><strong>Make following a time slot harder.</strong> Test slower acceleration and longer crossing times.</li>
<li><strong>Test delayed information.</strong> Check how the scheduling rules respond when communication arrives late.</li>
<li><strong>Change one rule at a time.</strong> Separate the effects of entry-speed prediction and the stop-line gate; compare MCTS with different rollout rules.</li>
</ol>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">What Implementing the Paper Taught Me</h2>
<p>I started with a question about how connected autonomous vehicles would exchange information and coordinate. Implementing FCFS gave me a simple version of that decision. Implementing the paper’s MCTS approach showed me how searching different orders and using heuristic rules could change the result.</p>
<p>Adding my own extension brought the safety question into focus: what should happen when a vehicle cannot do what the plan expects? That question matters even when the planned order looks efficient.</p>
<p>The GNN work added another lesson: defining the learning target matters. I kept the network structure but changed its task from predicting costs to selecting optimal candidates. The new model outperformed FCFS on average in the repeated comparison, while a direct old-versus-new model comparison remains necessary to isolate why.</p>
<p>The lesson I want to carry forward is to turn a paper’s idea into something I can inspect and test. I cannot conclude that MCTS always gives the lowest delay, or that adding a safety rule automatically proves a system safer. I can explain what I implemented, what happened in the comparison, and what still needs to be checked.</p>
</section>
<p class="study-source"><em>Charts show the original short-run tree-search comparison. The GNN table reports a separate five-seed experiment; differing protocols prevent a direct four-way ranking.</em></p>
</div>
