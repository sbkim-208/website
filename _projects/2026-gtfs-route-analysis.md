---
title: "GTFS Route-Structure Analysis"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "Parsed a GTFS feed with Pandas to compute route-level unique stop counts across multiple transit lines."
---

{% include project-study-style.html %}

<div id="project-study">
<header class="study-hero">
<span class="study-eyebrow">Transit data · table joins · route structure</span>
<h2>What does a transit feed tell us about a route?</h2>
<span class="study-badge">Initial analysis complete</span> <span class="study-badge amber">Synthetic GTFS feed</span>
<p>I wanted to pull a basic fact about a transit route directly from its raw feed: how many different stops does it serve? The answer was spread across several tables.</p>
<p>I worked through a synthetic GTFS feed with 3 routes and 16 stops. My question became: <strong>which joins turn stop-time records into a route-level description?</strong></p>
<p><strong>The stop-count stage is complete.</strong> Distance, stop spacing, mapping, and walking accessibility are still unfinished. This page separates the completed result from those next steps.</p>
</header>
<div class="study-metrics" aria-label="Project results at a glance">
<div class="study-metric"><span class="study-eyebrow">Routes analyzed</span><strong>3</strong><small>Unique stops counted for each route</small></div>
<div class="study-metric"><span class="study-eyebrow">Stops in the feed</span><strong>16</strong><small>Synthetic downtown transit network</small></div>
<div class="study-metric"><span class="study-eyebrow">Stops per route</span><strong>5–7</strong><small>Counts range from R10 to R12</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-scope">Study setup</a><a href="#study-process">How it works</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">Problem</h2>

<p>A GTFS feed technically contains everything you'd need to describe a route: stops, trip sequences, geometry. But none of it is handed to you pre-joined. The problem set was really about whether I could actually pull a basic structural fact, how many stops each route serves, out of the raw tables myself.</p>
<h3 id="study-scope">The feed used in this exercise</h3><p>The input is a synthetic downtown transit network with 3 routes and 16 stops; it is not Seoul-specific. The route labels belong to the exercise feed.</p>
<p>A stop can appear in multiple trips or routes. Counting unique stop IDs within each route avoids counting repeated stop-time records as additional stops.</p>

</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">Goals</h2>
<p>I wanted to answer these questions:</p><ul><li>Attach a route ID to each stop-time record.</li><li>Count unique stops for each route.</li><li>Keep the remaining distance, map, and accessibility tasks separate from completed work.</li></ul>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">Process</h2>

<h3>First, I connected trips to stop times</h3><p>I joined <code>stop_times</code> with <code>trips</code> to attach <code>route_id</code> to each stop time, then counted unique <code>stop_id</code>s per route.</p>

<h3>How the pieces fit together</h3><ol class="study-flow"><li><span>01 · READ</span><strong>Load the feed</strong>Inspect trips and stop_times.</li><li><span>02 · JOIN</span><strong>Attach each route</strong>Match stop times to their trip.</li><li><span>03 · GROUP</span><strong>Collect route stops</strong>Group records by route_id.</li><li><span>04 · COUNT</span><strong>Remove repeats</strong>Count distinct stop_id values.</li></ol>
</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">Results and Decisions</h2>

<p class="study-callout">The completed result is a route-level stop count. Distance, mapping, and accessibility remain next steps.</p>
<figure class="study-chart"><div class="study-chart-body"><h3>Unique stops by route</h3><p class="study-chart-unit">Unique stops · counted within each route</p><div class="study-chart-row"><span>R10</span><div class="study-chart-track"><span style="width:71.429%"></span></div><strong>5</strong></div><div class="study-chart-row"><span>R12</span><div class="study-chart-track"><span style="width:100.000%"></span></div><strong>7</strong></div><div class="study-chart-row"><span>R70</span><div class="study-chart-track"><span style="width:85.714%"></span></div><strong>6</strong></div></div><figcaption>Figure 1. A stop may belong to more than one route; these counts should not be summed to infer the number of distinct stops in the feed.</figcaption></figure><details><summary>View exact measurements</summary><div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">route_id</th><th scope="col">route_long_name</th><th scope="col">n_stops</th></tr></thead><tbody><tr><td>R10</td><td>Downtown ↔ SODO</td><td>5</td></tr><tr><td>R12</td><td>Magnolia ↔ Madison Park</td><td>7</td></tr><tr><td>R70</td><td>Downtown ↔ Northgate</td><td>6</td></tr></tbody></table></div></details>
<p>That's as far as I've gotten. The distance/spacing/map/walkshed parts are still scaffolded, not implemented.</p>

</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">Limitations</h2>
<ul><li><strong>Synthetic input.</strong> These counts do not establish coverage on a real transit system.</li><li><strong>Coverage is not geometry.</strong> Unique stop counts do not measure route length or spacing.</li><li><strong>Accessibility remains unfinished.</strong> No walkshed result is claimed.</li></ul>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">Where I'm Stopping, and What Comes Next</h2>
<p class="study-callout amber">I am stopping this write-up at the completed route-level stop counts. The remaining problem-set tasks are listed below as future work.</p><p>If I return to the project, I would prioritize:</p><ol class="study-roadmap"><li><strong>Calculate distances.</strong> Use a projected coordinate system for per-segment and route lengths.</li><li><strong>Measure stop spacing.</strong> Relate distance to the route sequence.</li><li><strong>Build the map.</strong> Display the routes and stops in Folium.</li><li><strong>Explore the optional walkshed.</strong> Complete the 400 m walking-accessibility exercise.</li></ol>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">Reflection</h2>
<p>I realized a GTFS feed looks complete but isn't usable until you actually do the joins yourself. Even a basic fact like stop count per route took deciding which tables to combine and how, and that step is easy to underestimate before you've done it once.</p>
</section>
<p class="study-source"><em>Charts summarize the measurements recorded in this project write-up. They do not represent new experiments.</em></p>
</div>
