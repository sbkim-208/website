---
title: "LSTM & GCN vs. Classical Baselines"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "From-scratch LSTM and GCN implementations in PyTorch, each held to the same bar: beat a classical baseline (Random Forest, MLP) on the identical task and split, or don't claim the win."
---

{% include project-study-style.html %}

<div id="project-study">
<header class="study-hero">
<span class="study-eyebrow">Machine learning · sequence forecasting · graph structure</span>
<h2>When does a more complex model earn its place?</h2>
<span class="study-badge">Baseline comparison complete</span> <span class="study-badge amber">Small-scale experiments</span>
<p>I wanted to know when a neural network adds value over a simpler baseline. A strong score on its own could not answer that question.</p>
<p>I built an LSTM and a GCN in PyTorch and tested each against a baseline on the same task and split. My question became: <strong>does the architecture capture information that the baseline misses?</strong></p>
<p><strong>The two small comparisons are complete.</strong> Random Forest won the sequence task, while the GCN won a synthetic task designed around neighboring-node information. The results point to different roles for data structure and model complexity.</p>
</header>
<div class="study-metrics" aria-label="Project results at a glance">
<div class="study-metric"><span class="study-eyebrow">Sequence baseline</span><strong>8.19</strong><small>Random Forest test RMSE; LSTM: 14.88</small></div>
<div class="study-metric"><span class="study-eyebrow">Graph prediction</span><strong>0.733</strong><small>GCN R²; MLP: −4.25</small></div>
<div class="study-metric"><span class="study-eyebrow">Graph split</span><strong>8 / 8</strong><small>Labeled training nodes / evaluation nodes</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-scope">Study setup</a><a href="#study-process">How it works</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">Problem</h2>

<p>It's easy to report a deep-learning R² by itself and call it a result. It's a lot harder to trust once you put it next to what a Random Forest or a plain MLP gets on the same data. I wanted to know when a fancier model is actually earning its keep and when it's just a fancier way of doing worse.</p>
<h3 id="study-scope">The two experiment settings</h3><p>The sequence experiment predicts next-hour bike-share demand from a 24-hour input window. The LSTM receives the sequence; Random Forest receives the same values flattened into lag inputs.</p>
<p>The graph experiment uses 16 synthetic transit nodes, with 8 labeled training nodes and 8 evaluation nodes. The target depends on neighboring stations’ transfer-count and parking values.</p>

</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">Goals</h2>
<p>I wanted to answer these questions:</p><ul><li>Would an LSTM beat Random Forest using the same 24-hour input windows?</li><li>Would increasing the LSTM’s hidden size close the gap?</li><li>Would a GCN outperform an MLP when the synthetic target depends on neighbors?</li></ul>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">Process</h2>

<h3>First, I compared the sequence models</h3><p>For the sequence task, I trained a single-layer LSTM (<code>hidden=32</code>) to predict next-hour bike-share demand from a 24-hour sliding window, with no hand-built lag or calendar features, so it has to learn the daily cycle from the raw sequence alone. The Random Forest baseline got the same windows, just flattened into 24 lag features. When RF won clearly, I didn't want to just accept that, so I ran a follow-up hidden-size sweep on the LSTM to rule out the boring explanation, that it was under-capacity rather than the wrong tool for this amount of data.</p>
<h3>Then, I tested a neighbor-dependent target</h3><p>For the node task, I built a 16-node synthetic transit graph, a main line plus 2 branches and a cross-connection, and generated the ridership target so it depends on a station's neighbors' transfer-count and parking values, not just its own. An MLP can't solve that no matter how it's trained. A 2-layer GCN (<code>H' = ReLU(Â·H·W)</code>) and a plain MLP were trained on the same 8 labeled nodes and evaluated on the other 8.</p>

<h3>How the pieces fit together</h3><ol class="study-flow"><li><span>01 · DEFINE</span><strong>Fix the task</strong>Specify the prediction target.</li><li><span>02 · MATCH</span><strong>Share the split</strong>Keep evaluation examples consistent.</li><li><span>03 · TRAIN</span><strong>Fit both models</strong>Compare neural and baseline models.</li><li><span>04 · CHECK</span><strong>Explain the gap</strong>Inspect capacity and available information.</li></ol>
</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">Results and Decisions</h2>

<p class="study-callout">Random Forest performed best on the sequence task. The GCN performed best on the synthetic task where the target depended on neighboring nodes.</p>
<p>First, RF beat the LSTM, clearly:</p>
<figure class="study-chart"><div class="study-chart-body"><h3>Sequence prediction error</h3><p class="study-chart-unit">Test RMSE · lower is better</p><div class="study-chart-row"><span>LSTM</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>14.88</strong></div><div class="study-chart-row"><span>Random Forest</span><div class="study-chart-track"><span style="width:55.040%"></span></div><strong>8.19</strong></div></div><figcaption>Figure 1. Initial sequence comparison using the same 24-hour windows.</figcaption></figure><details><summary>View exact measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Model</th><th scope="col">MAE</th><th scope="col">RMSE</th><th scope="col">R²</th></tr></thead><tbody><tr><td>LSTM (raw sequence, no lag features)</td><td>9.76</td><td>14.88</td><td>0.884</td></tr><tr><td>Random Forest (24 flattened lag features)</td><td>5.04</td><td>8.19</td><td><strong>0.965</strong></td></tr></tbody></table></div></details>
<p>The capacity sweep I ran afterward:</p>
<figure class="study-chart"><div class="study-chart-body"><h3>LSTM hidden-size sweep</h3><p class="study-chart-unit">Test RMSE · lower is better</p><div class="study-chart-row"><span>8 units</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>17.02</strong></div><div class="study-chart-row"><span>16 units</span><div class="study-chart-track"><span style="width:91.363%"></span></div><strong>15.55</strong></div><div class="study-chart-row"><span>32 units</span><div class="study-chart-track"><span style="width:89.894%"></span></div><strong>15.3</strong></div><div class="study-chart-row"><span>64 units</span><div class="study-chart-track"><span style="width:76.204%"></span></div><strong>12.97</strong></div></div><figcaption>Figure 2. Separate follow-up runs. Increasing hidden size did not close the gap to the Random Forest’s 8.19.</figcaption></figure><details><summary>View exact measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Hidden size</th><th scope="col">Test RMSE</th></tr></thead><tbody><tr><td>8</td><td>17.02</td></tr><tr><td>16</td><td>15.55</td></tr><tr><td>32</td><td>15.30</td></tr><tr><td>64</td><td>12.97</td></tr></tbody></table></div></details>
<p>More capacity kept helping, but even at 64 units the LSTM's 12.97 doesn't get close to RF's 8.19. Increasing capacity alone did not close the gap in the settings I tried. The Random Forest, using the same windows flattened into lag inputs, remained ahead.</p>
<p>Second, on the node task, it went the other way:</p>
<figure class="study-chart"><div class="study-chart-body"><h3>Node prediction error</h3><p class="study-chart-unit">MAE · lower is better</p><div class="study-chart-row"><span>MLP</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>105.94</strong></div><div class="study-chart-row"><span>GCN</span><div class="study-chart-track"><span style="width:26.383%"></span></div><strong>27.95</strong></div></div><figcaption>Figure 3. Synthetic neighbor-dependent target with 8 training and 8 evaluation nodes.</figcaption></figure><details><summary>View exact measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">Model</th><th scope="col">MAE</th><th scope="col">R²</th></tr></thead><tbody><tr><td>MLP (no graph)</td><td>105.94</td><td><strong>−4.25</strong></td></tr><tr><td>GCN (uses <code>Â</code>)</td><td>27.95</td><td><strong>0.733</strong></td></tr></tbody></table></div></details>
<p>The MLP did worse than just predicting the mean, since it can't see neighbor information at all. The GCN propagates it through the normalized adjacency matrix and fits well.</p>
<p>RF beat the LSTM because its lag features already captured the sequence dependence a small model couldn't learn on its own. The GCN beat the MLP because neighbor structure was the only thing that could explain the target in the first place. Knowing which case I was in before picking a model was really the point of both experiments.</p>

</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">Limitations</h2>
<ul><li><strong>Limited sequence experiment.</strong> The conclusion applies to this dataset and split.</li><li><strong>A capacity sweep is not exhaustive tuning.</strong> Other training choices could change the LSTM result.</li><li><strong>A deliberately synthetic graph target.</strong> Only 16 nodes were used, and the target was constructed to depend on neighbors.</li><li><strong>Different runs should remain separate.</strong> The initial LSTM RMSE is 14.88; the later 32-unit sweep run reports 15.30.</li></ul>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">Where I'm Stopping, and What Comes Next</h2>
<p class="study-callout amber">These comparisons show why I need a matched baseline before interpreting a model score. They do not establish a general ranking of neural and classical methods.</p><p>If I return to the project, I would prioritize:</p><ol class="study-roadmap"><li><strong>Repeat the comparisons.</strong> Use multiple seeds and evaluation periods.</li><li><strong>Broaden the sequence experiment.</strong> Test additional training settings while retaining the matched baseline.</li><li><strong>Use observed transit data.</strong> Check whether graph information helps beyond the synthetic construction.</li></ol>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">Reflection</h2>
<p>I realized that a more complex architecture doesn't automatically win. It only helps when it actually matches how the underlying data is structured, and it can lose badly when it doesn't. I also learned that a capacity sweep is a cheap way to check whether a larger model closes the gap, while leaving other training choices open.</p>
</section>
<p class="study-source"><em>Charts summarize the measurements recorded in this project write-up. They do not represent new experiments.</em></p>
</div>
