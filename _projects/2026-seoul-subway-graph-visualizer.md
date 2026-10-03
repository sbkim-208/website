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

{% include project-study-style.html %}

<div id="project-study">
<header class="study-hero">
<span class="study-eyebrow">Transit networks · graph search · interactive visualization</span>
<h2>How much search can a useful heuristic save?</h2>
<span class="study-badge">Visualizer implemented</span> <span class="study-badge amber">One route comparison</span>
<p>I wanted to understand why one graph-search algorithm explores less of a transport network than another. Writing the algorithms was only part of that question; I also wanted to see the search unfold.</p>
<p>I built a visualizer over a roughly 50-station Seoul subway graph. My question became: <strong>can a useful heuristic find the same route while visiting fewer stations?</strong></p>
<p><strong>The visualizer and one route comparison are complete.</strong> A* visited 7 nodes on Gangnam → Hapjeong, compared with Dijkstra’s 40. This is a result for one query, not a network-wide benchmark.</p>
</header>
<div class="study-metrics" aria-label="Project results at a glance">
<div class="study-metric"><span class="study-eyebrow">A* search</span><strong>7</strong><small>Nodes visited on Gangnam → Hapjeong</small></div>
<div class="study-metric"><span class="study-eyebrow">Dijkstra search</span><strong>40</strong><small>Nodes visited on the same query</small></div>
<div class="study-metric"><span class="study-eyebrow">Shared route cost</span><strong>26</strong><small>All four methods returned 6 hops</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-scope">Study setup</a><a href="#study-process">How it works</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">Problem</h2>

<p>I wanted to sharpen my understanding of search algorithms on a real transportation network, not just implement BFS, DFS, Dijkstra, and A*, but actually be able to say why one beats another and by how much, on a real graph instead of a whiteboard example.</p>
<h3 id="study-scope">The network and comparison</h3><p>The graph contains roughly 50 Seoul subway stations. Each algorithm receives the same start and goal, while the frontend shows its intermediate search states.</p>
<p>The reported benchmark is Gangnam → Hapjeong. All four methods returned 6 hops and a total cost of 26 for that query.</p>
<figure><img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search animating over the Seoul subway graph, then an algorithm-comparison table" style="max-width:100%;"><figcaption>A* search animating over the Seoul subway graph, then an algorithm-comparison table.</figcaption></figure>
</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">Goals</h2>
<p>I wanted to answer these questions:</p><ul><li>Could every algorithm share a single interface and the same visualizer?</li><li>Would A* reduce the number of visited nodes on the same route query?</li><li>Would the visualization remain readable on the full station graph?</li></ul>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">Process</h2>

<h3>First, I checked the heuristic</h3><p>Before writing <code>a_star.py</code> I looked at a shortcut: precompute costs with Dijkstra once and reuse that table as A*'s heuristic. It turned out that isn't safe for an arbitrary query, since the table only lower-bounds the cost for the specific pairs it was built from, so I used straight-line distance instead, which needs no precomputation at all.</p>
<h3>Then, I separated search from the display</h3><p>Every algorithm shares one interface and streams step-by-step frames, so the frontend can render any of them without knowing which one it is. A new algorithm gets auto-discovered at startup, which is how I added A* later without touching the frontend at all.</p>
<h3>Finally, I tested the full graph</h3><p>The placeholder styling that looked fine on small demo graphs broke once I loaded the real ~50-station graph. Labels overlapped and were hard to read, so I fixed the font, label size, node colors, and layout spacing until it held up.</p>
<p>Backend is FastAPI + WebSocket, frontend is React + TypeScript with React Flow for the graph canvas. <code>/eval</code> runs every algorithm on the same start/goal and compares them; <code>/ws/run</code> streams a single run. Algorithms: BFS, DFS, Dijkstra, and A*, plus a few more I added while working through the session's homework problems against the same interface (Number of Islands, Course Schedule, Shortest Path in Binary Matrix, Network Delay Time).</p>

<h3>How the pieces fit together</h3><ol class="study-flow"><li><span>01 · QUERY</span><strong>Choose a route</strong>Use the same start and goal.</li><li><span>02 · SEARCH</span><strong>Advance one step</strong>Run the selected graph algorithm.</li><li><span>03 · STREAM</span><strong>Send the frame</strong>WebSocket carries the intermediate state.</li><li><span>04 · COMPARE</span><strong>Inspect the result</strong>Compare route cost and visited nodes.</li></ol>
</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">Results and Decisions</h2>

<p class="study-callout">A* visited 7 nodes versus Dijkstra’s 40 on this query. That is evidence for this route, not a guarantee of the same reduction across the network.</p>
<p>I ran all four algorithms on the same Gangnam → Hapjeong route through <code>/eval</code>, to check the A* payoff was real and not just assumed:</p>
<figure class="study-chart"><div class="study-chart-body"><h3>Search effort on Gangnam → Hapjeong</h3><p class="study-chart-unit">Nodes visited · lower is fewer explored nodes</p><div class="study-chart-row"><span>A*</span><div class="study-chart-track"><span style="width:17.500%"></span></div><strong>7</strong></div><div class="study-chart-row"><span>DFS</span><div class="study-chart-track"><span style="width:62.500%"></span></div><strong>25</strong></div><div class="study-chart-row"><span>BFS</span><div class="study-chart-track"><span style="width:67.500%"></span></div><strong>27</strong></div><div class="study-chart-row"><span>Dijkstra</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>40</strong></div></div><figcaption>Figure 1. All four returned cost 26 and 6 hops. This comparison covers one route query.</figcaption></figure><details><summary>View exact measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Algorithm</th><th scope="col">Hops</th><th scope="col">Total Cost</th><th scope="col">Nodes Visited</th><th scope="col">Iterations</th></tr></thead><tbody><tr><td>A*</td><td>6</td><td>26</td><td><strong>7</strong></td><td>7</td></tr><tr><td>DFS</td><td>6</td><td>26</td><td>25</td><td>25</td></tr><tr><td>BFS</td><td>6</td><td>26</td><td>27</td><td>27</td></tr><tr><td>Dijkstra</td><td>6</td><td>26</td><td>40</td><td>40</td></tr></tbody></table></div></details>
<p>All four returned the same cost and hop count on this query, while visiting very different amounts of the graph. A* visited only 7 of the ~50 stations, against Dijkstra's 40, because the heuristic keeps pulling the search toward Hapjeong instead of expanding outward evenly. I don't think DFS's 25 means much beyond this one route though. It's probably just this route's neighbor-list ordering happening to point roughly the right way, since nothing in DFS actually biases it toward the goal.</p>

</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">Limitations</h2>
<ul><li><strong>One route query.</strong> The reported result does not describe all station pairs.</li><li><strong>Equal costs do not imply equal guarantees.</strong> BFS and DFS are not generally optimal for weighted routing.</li><li><strong>The heuristic needs compatible costs.</strong> Straight-line distance must lower-bound the actual routing objective.</li></ul>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">Where I'm Stopping, and What Comes Next</h2>
<p class="study-callout amber">The current result shows a clear difference in visited nodes for one query. Broader routing and runtime comparisons remain future work.</p><p>If I return to the project, I would prioritize:</p><ol class="study-roadmap"><li><strong>Expand the route set.</strong> Compare multiple start–goal pairs.</li><li><strong>Check heuristic bounds.</strong> Verify them against the graph’s edge-cost definition.</li><li><strong>Measure runtime and ordering effects.</strong> Add timing and vary DFS neighbor order.</li></ol>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">Reflection</h2>
<p>I realized that a heuristic is only as good as the guarantee behind it. It was tempting to reuse a precomputed cost table because it looked like free performance, and I only avoided that mistake by working through whether it was actually admissible first. I also learned that a design choice isn't proven until it's measured on the real graph, not the small demo one.</p>
</section>
<p class="study-source"><em>Charts summarize the measurements recorded in this project write-up. They do not represent new experiments.</em></p>
</div>
