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
<p>I built an interactive graph-algorithm simulator over a ~50-station Seoul subway graph. A FastAPI + WebSocket backend runs a search algorithm step by step and streams each intermediate state to a React/TypeScript frontend I wrote to render it live on a node-link canvas.</p>
</header>
<div class="study-metrics" aria-label="Project results at a glance">
<div class="study-metric"><span class="study-eyebrow">A* search</span><strong>7</strong><small>Nodes visited on Gangnam → Hapjeong</small></div>
<div class="study-metric"><span class="study-eyebrow">Dijkstra search</span><strong>40</strong><small>Nodes visited on the same query</small></div>
<div class="study-metric"><span class="study-eyebrow">Shared route cost</span><strong>26</strong><small>All four methods returned 6 hops</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-goals">Goals</a><a href="#study-process">How it works</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a><a href="#study-reflection">Reflection</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">Problem</h2>
<p>I wanted to sharpen my understanding of search algorithms on a real transportation network, not just implement BFS, DFS, Dijkstra, and A*, but actually be able to say why one beats another and by how much, on a real graph instead of a whiteboard example.</p>
</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">Goals</h2>
<p>I wanted every algorithm to share one interface, so adding a new one later wouldn't mean touching the frontend at all. And I didn't want to just assume A* would win because it's supposed to. I wanted to actually measure the payoff of each design decision, on the real ~50-station graph, not a toy one.</p>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">Process</h2>
<p>Before writing <code>a_star.py</code> I looked at a shortcut: precompute costs with Dijkstra once and reuse that table as A*'s heuristic. It turned out that isn't safe for an arbitrary query, since the table only lower-bounds the cost for the specific pairs it was built from, so I used straight-line distance instead, which needs no precomputation at all.</p>
<p>Every algorithm shares one interface and streams step-by-step frames, so the frontend can render any of them without knowing which one it is. A new algorithm gets auto-discovered at startup, which is how I added A* later without touching the frontend at all.</p>
<p>The placeholder styling that looked fine on small demo graphs broke once I loaded the real ~50-station graph. Labels overlapped and were hard to read, so I fixed the font, label size, node colors, and layout spacing until it held up.</p>
<p>Backend is FastAPI + WebSocket, frontend is React + TypeScript with React Flow for the graph canvas. <code>/eval</code> runs every algorithm on the same start/goal and compares them; <code>/ws/run</code> streams a single run. Algorithms: BFS, DFS, Dijkstra, and A*, plus a few more I added while working through the session's homework problems against the same interface (Number of Islands, Course Schedule, Shortest Path in Binary Matrix, Network Delay Time).</p>
<figure><img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search animating over the Seoul subway graph, then an algorithm-comparison table" style="max-width:100%;"><figcaption>A* search animating over the Seoul subway graph, then an algorithm-comparison table.</figcaption></figure>
</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">Results and Decisions</h2>
<p class="study-callout">A* visited 7 nodes versus Dijkstra’s 40 on this query. That is evidence for this route, not a guarantee of the same reduction across the network.</p>
<p>I ran all four algorithms on the same Gangnam → Hapjeong route through <code>/eval</code>, to check the A* payoff was real and not just assumed:</p>
<div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Algorithm</th><th scope="col">Hops</th><th scope="col">Total Cost</th><th scope="col">Nodes Visited</th><th scope="col">Iterations</th></tr></thead><tbody><tr><td>A*</td><td>6</td><td>26</td><td><strong>7</strong></td><td>7</td></tr><tr><td>DFS</td><td>6</td><td>26</td><td>25</td><td>25</td></tr><tr><td>BFS</td><td>6</td><td>26</td><td>27</td><td>27</td></tr><tr><td>Dijkstra</td><td>6</td><td>26</td><td>40</td><td>40</td></tr></tbody></table></div>
<p>All four returned the same cost and hop count on this query, while visiting very different amounts of the graph. A* visited only 7 of the ~50 stations, against Dijkstra's 40, because the heuristic keeps pulling the search toward Hapjeong instead of expanding outward evenly. I don't think DFS's 25 means much beyond this one route though. It's probably just this route's neighbor-list ordering happening to point roughly the right way, since nothing in DFS actually biases it toward the goal.</p>
</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">Limitations</h2>
<p>The reported comparison covers one start–goal pair on a roughly 50-station graph. Equal route costs on this query do not make BFS or DFS optimal for weighted routing in general. A* also needs a heuristic that lower-bounds the actual edge-cost objective; using straight-line distance requires compatible cost units and assumptions.</p>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">What Comes Next</h2>
<p class="study-callout amber">A useful next comparison would cover more start–goal pairs, verify the heuristic against the graph’s cost definition, and report runtime as well as visited nodes. Different neighbor orderings would help show how sensitive DFS is to traversal order.</p>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">Reflection</h2>
<p>I realized that a heuristic is only as good as the guarantee behind it. It was tempting to reuse a precomputed cost table because it looked like free performance, and I only avoided that mistake by working through whether it was actually admissible first. I also learned that a design choice isn't proven until it's measured on the real graph, not the small demo one.</p>
</section>
</div>
