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
<p>I worked through a GTFS problem set, 3 routes, 16 stops, a synthetic feed modeled on a generic downtown transit network, not Seoul-specific, to characterize route structure directly from the raw feed instead of trusting a route map at face value.</p>
</header>
<div class="study-metrics" aria-label="Project results at a glance">
<div class="study-metric"><span class="study-eyebrow">Routes analyzed</span><strong>3</strong><small>Unique stops counted for each route</small></div>
<div class="study-metric"><span class="study-eyebrow">Stops in the feed</span><strong>16</strong><small>Synthetic downtown transit network</small></div>
<div class="study-metric"><span class="study-eyebrow">Stops per route</span><strong>5–7</strong><small>Counts range from R10 to R12</small></div>
</div>
<nav class="study-nav" aria-label="Case study sections"><a href="#study-problem">The question</a><a href="#study-goals">Goals</a><a href="#study-process">How it works</a><a href="#study-results">Results</a><a href="#study-limitations">Limitations</a><a href="#study-next">Next steps</a><a href="#study-reflection">Reflection</a></nav>
<section id="study-problem" class="study-section" aria-labelledby="study-title-problem">
<span class="study-eyebrow">01 / Case study</span>
<h2 id="study-title-problem">Problem</h2>
<p>A GTFS feed technically contains everything you'd need to describe a route: stops, trip sequences, geometry. But none of it is handed to you pre-joined. The problem set was really about whether I could actually pull a basic structural fact, how many stops each route serves, out of the raw tables myself.</p>
</section>
<section id="study-goals" class="study-section" aria-labelledby="study-title-goals">
<span class="study-eyebrow">02 / Case study</span>
<h2 id="study-title-goals">Goals</h2>
<p>The immediate goal was just unique stop counts per route. The problem set goes further than that though: per-segment distance in a projected CRS, route length, average stop spacing, an interactive Folium map, and a 400m-walkshed accessibility bonus. I haven't gotten to any of that part yet.</p>
</section>
<section id="study-process" class="study-section" aria-labelledby="study-title-process">
<span class="study-eyebrow">03 / Case study</span>
<h2 id="study-title-process">Process</h2>
<p>I joined <code>stop_times</code> with <code>trips</code> to attach <code>route_id</code> to each stop time, then counted unique <code>stop_id</code>s per route.</p>

</section>
<section id="study-results" class="study-section" aria-labelledby="study-title-results">
<span class="study-eyebrow">04 / Case study</span>
<h2 id="study-title-results">Results and Decisions</h2>
<p class="study-callout">The completed result is a route-level stop count. Distance, mapping, and accessibility remain next steps.</p>
<div class="study-table" role="region" aria-label="Experiment results" tabindex="0"><table><thead><tr><th scope="col">route_id</th><th scope="col">route_long_name</th><th scope="col">n_stops</th></tr></thead><tbody><tr><td>R10</td><td>Downtown ↔ SODO</td><td>5</td></tr><tr><td>R12</td><td>Magnolia ↔ Madison Park</td><td>7</td></tr><tr><td>R70</td><td>Downtown ↔ Northgate</td><td>6</td></tr></tbody></table></div>
<p>That's as far as I've gotten. The distance/spacing/map/walkshed parts are still scaffolded, not implemented.</p>
</section>
<section id="study-limitations" class="study-section" aria-labelledby="study-title-limitations">
<span class="study-eyebrow">05 / Case study</span>
<h2 id="study-title-limitations">Limitations</h2>
<p>The feed is synthetic and is not a model of Seoul. Unique stop counts describe route coverage, but do not establish route length, stop spacing, or walking accessibility. Those parts of the problem set remain unimplemented.</p>
</section>
<section id="study-next" class="study-section" aria-labelledby="study-title-next">
<span class="study-eyebrow">06 / Case study</span>
<h2 id="study-title-next">What Comes Next</h2>
<p class="study-callout amber">Complete the remaining distance and spacing calculations in a projected coordinate system, then build the Folium map and the optional 400 m walkshed analysis.</p>
</section>
<section id="study-reflection" class="study-section study-reflection" aria-labelledby="study-title-reflection">
<span class="study-eyebrow">07 / Case study</span>
<h2 id="study-title-reflection">Reflection</h2>
<p>I realized a GTFS feed looks complete but isn't usable until you actually do the joins yourself. Even a basic fact like stop count per route took deciding which tables to combine and how, and that step is easy to underestimate before you've done it once.</p>
</section>
</div>
