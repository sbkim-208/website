---
title: "FDR Drive Northbound: Could I Warn a Driver Before Exit 2?"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I tested whether I could warn about congestion on northbound FDR segment 217 while a driver still had time to take Exit 2. The Exit 2 demo replays 2026 road predictions and lets you change when a warning would reach the driver."
thumbnail: /assets/img/projects/fdr-217-exit-replay.svg
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

<style>
#content:has(#fdr-study){max-width:1100px}
#fdr-study{--ink:#17374a;--sub:#526b7a;--teal:#087e80;--line:#dbe5ea;color:var(--ink);background:#f3f7f8;border:1px solid var(--line);border-radius:18px;padding:24px;margin-top:24px;font-size:16px;line-height:1.75}
#fdr-study *{box-sizing:border-box}
#fdr-study a{color:#076c73}
#fdr-study a:focus-visible,#fdr-study summary:focus-visible{outline:3px solid #ad6a18;outline-offset:4px}
#fdr-study .fdr-hero{background:#17374a;color:#f4fafb;border-radius:12px;padding:32px;margin:0 0 20px}
#fdr-study .fdr-hero p{color:#d9e9ee}
#fdr-study .fdr-hero h2{color:white;border:0;font-size:29px;line-height:1.25;margin:14px 0 20px;padding:0;letter-spacing:-.6px}
#fdr-study .fdr-eyebrow{font-size:11px;font-weight:750;letter-spacing:1.5px;text-transform:uppercase;color:var(--teal)}
#fdr-study .fdr-hero .fdr-eyebrow{color:#aadbd8}
#fdr-study .fdr-metrics{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:12px;margin:20px 0}
#fdr-study .fdr-metric{background:white;border:1px solid var(--line);border-radius:10px;padding:20px}
#fdr-study .fdr-metric strong{display:block;font-size:35px;line-height:1.2;color:var(--ink);margin:10px 0}
#fdr-study .fdr-metric small{display:block;color:var(--sub);line-height:1.5;font-size:13px}
#fdr-study .fdr-nav{display:flex;gap:8px;flex-wrap:wrap;padding:12px 0 22px}
#fdr-study .fdr-nav a{padding:7px 12px;border:1px solid var(--line);border-radius:6px;background:white;font-size:13px;text-decoration:none}
#fdr-study .fdr-nav a:hover{background:#e5f1ee}
#fdr-study .fdr-section{background:white;border:1px solid var(--line);border-radius:12px;padding:28px;margin:0 0 20px;scroll-margin-top:90px}
#fdr-study h2{font-size:25px;line-height:1.3;border:0;margin:8px 0 20px;padding:0;color:var(--ink)}
#fdr-study h3{font-size:19px;margin:28px 0 10px;line-height:1.4}
#fdr-study p{margin:12px 0}
#fdr-study figure{margin:22px 0!important;border:1px solid var(--line);border-radius:10px;overflow:hidden;background:white}
#fdr-study figure img{display:block;width:100%;height:auto}
#fdr-study figcaption{padding:14px 18px;font-size:13px;line-height:1.65;color:var(--sub);background:#f5f8fa;border-top:1px solid var(--line)}
#fdr-study .fdr-source{font-size:12px;color:var(--sub);overflow-wrap:anywhere}
@media(max-width:650px){#fdr-study{padding:12px;margin-left:-8px;margin-right:-8px}#fdr-study .fdr-section,#fdr-study .fdr-hero{padding:20px 16px}#fdr-study .fdr-metrics{grid-template-columns:1fr}#fdr-study .fdr-metric{padding:16px}#fdr-study .fdr-metric strong{font-size:30px}#fdr-study .fdr-flow{grid-template-columns:1fr 1fr}#fdr-study h2,#fdr-study .fdr-hero h2{font-size:24px}#fdr-study figcaption{padding:12px;font-size:12px}}
@media(prefers-color-scheme:dark){#fdr-study{background:#14222b;--ink:#e2eff3;--sub:#b6c9d2;--line:#34505e;--teal:#8ad4cc}#fdr-study .fdr-section,#fdr-study .fdr-metric,#fdr-study .fdr-nav a{background:#1b303b}#fdr-study a{color:#8ad4cc}#fdr-study .fdr-flow li,#fdr-study .fdr-roadmap li,#fdr-study th{background:#243f4b}#fdr-study figcaption{background:#1b303b}#fdr-study .fdr-callout{background:#203d3b}#fdr-study .fdr-callout.amber{background:#493a25}#fdr-study .fdr-nav a:hover{background:#284a50}}
</style>
<div id="fdr-study">
<header class="fdr-hero"><span class="fdr-eyebrow">FDR Drive northbound · Offline replay</span><h2>Could an earlier warning have changed my commute?</h2>
<p>During my commute to work, I sometimes found out about traffic only after I’d passed the turn where I could have taken another route. I wanted to know whether road speeds could tell me that a slowdown was coming while I still had a choice.</p>
<p>I narrowed the study to one stretch of New York’s FDR Drive: northbound segment 217, from Catherine Slip to 25th Street. Exit 2 is the turn a driver would have to take before reaching that stretch.</p></header>
<nav class="fdr-nav" aria-label="Case study sections"><a href="#fdr-data">NYC sensor data</a><a href="#fdr-method">Choosing the warning</a><a href="#fdr-2026">2026 results</a><a href="#fdr-exit-demo">Try the simulator</a></nav>
<section id="fdr-data" class="fdr-section" aria-labelledby="fdr-title-data"><span class="fdr-eyebrow">April–July 2024 · Five boroughs</span><h2 id="fdr-title-data">First, I checked the data</h2>
<p>I started with 4,233,169 NYC DOT traffic sensor records. Before using them to predict a slowdown, I wanted to know how often the sensors returned a usable reading. In Queens, 93.4% of records had no <code>-101</code> error flag. In Manhattan, where the FDR runs, it was only 56.3%.</p>
<figure><img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Records without the error flag: Queens 93.4%, Bronx 83.7%, Staten Island 65.1%, Brooklyn 62.1%, Manhattan 56.3%." loading="lazy"><figcaption>Share of recorded readings without the error flag in each borough. This measures the quality of the records I received; it doesn’t count readings that never arrived.</figcaption></figure>
<p>Those gaps changed how I tested a warning. At first I looked six rows ahead to find the speed 30 minutes later. But if a sensor skipped a reading, the sixth row was too late. For a 9:00 prediction, I checked the timestamp for 9:30. If a future reading was missing, I left it missing instead of filling in the answer.</p>
<p><a href="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}">Explore the original NYC map of sensor errors and evening congestion</a> · <a href="{{ '/projects/2026-nyc-traffic-analysis/original-study/' | relative_url }}">Read the full earlier analysis, with the original charts</a>.</p></section>
<section id="fdr-method" class="fdr-section" aria-labelledby="fdr-title-method"><span class="fdr-eyebrow">From citywide records to segment 217</span><h2 id="fdr-title-method">What would I want to know before the exit?</h2>
<p>My first model predicted speed exactly 30 minutes ahead. That missed a case I cared about: traffic could slow after 10 minutes and recover by minute 30. So I changed the question. While the road was still flowing, would a slowdown <em>start</em> at any point in the next half hour?</p>
<p>I compared models using 2024 data, then checked the warnings against 2025 events. I also tried waiting for five minutes of recovery before allowing another alert. On segment 217, that cut the number of events caught at least 10 minutes early from 99 to 86 out of the same 501 events. I kept the original 30-minute repeat-alert rule.</p></section>
<section id="fdr-2026" class="fdr-section" aria-labelledby="fdr-title-2026"><span class="fdr-eyebrow">January–March 2026 · Weekdays</span><h2 id="fdr-title-2026">What happened after the alerts?</h2>
<p>I used a random forest that gave more weight during training to slowdowns that began within 5 or 10 minutes. I kept the settings fixed for the replay. I had already looked at this quarter in earlier work.</p>
<p>On segment 217, the model warned about 101 of 290 events before they began. Only 53 got at least 10 minutes’ notice. Looking instead at the 137 alerts the model issued, I could match 101 to a later slowdown. There were 15 false alarms and 21 alerts I couldn’t score because road observations were missing.</p>
<div class="fdr-metrics" aria-label="January–March 2026, northbound FDR segment 217, weekday offline evaluation">
<div class="fdr-metric"><span class="fdr-eyebrow">2026 · Segment 217</span><strong>101 / 290</strong><small>Congestion events detected<br>5–30 minutes in advance</small></div>
<div class="fdr-metric"><span class="fdr-eyebrow">At least 10 minutes ahead</span><strong>53 / 290</strong><small>Events with an earlier warning<br>among the same 290 events</small></div>
<div class="fdr-metric"><span class="fdr-eyebrow">Missing observations</span><strong>21 / 137</strong><small>Road alerts whose outcomes<br>I could not verify</small></div></div>
<h3>When could the model make a prediction?</h3>
<p>For the 2026 replay I checked every scheduled five-minute reading on segment 217. Of 18,432 checks, 1,738 showed the road was flowing. The model could score 704 of those. The other 1,034 lacked the recent readings it needed. I show those separately in the simulator because I wouldn’t want someone to read silence as an all-clear.</p>
<p>I ran the saved model again on all 704 inputs in the original feature order. Every score matched the replay within 10<sup>−12</sup>. <a href="{{ '/assets/interactive/fdr-model-check.json' | relative_url }}">View the model version and monthly counts</a>.</p></section>
<section id="fdr-exit-demo" class="fdr-section" aria-labelledby="fdr-title-demo"><span class="fdr-eyebrow">Exit 2 · Interactive replay</span><h2 id="fdr-title-demo">Would I still have time to take the exit?</h2>
<p>On January 26, 2026, at 11:50 p.m., segment 217 was still flowing. The model issued a warning in the replay, and five minutes later the recorded speed fell below the congestion threshold. Those five minutes tell me how far ahead the model saw the slowdown. They don’t tell me how far a driver was from the exit.</p>
<p>Try giving the driver 60 seconds to reach Exit 2. With a 10-second notification delay, the warning appears with 50 seconds left. Change the delay to 90 seconds and the driver has already passed the exit. The road prediction and the later road speed come from the study records. You choose the route, time to the exit, and display delay. I didn’t measure a phone’s delivery time or any time saved by taking the detour.</p>
<figure><iframe id="fdr-replay-frame" title="Interactive Exit 2 warning simulator with recorded road alerts and adjustable journey timings" src="{{ '/assets/interactive/fdr-exit-2-replay.html' | relative_url }}" loading="lazy" style="display:block;width:100%;height:850px;border:0"></iframe>
<figcaption>The road records are from weekday observations in the first quarter of 2026. <a href="{{ '/assets/interactive/fdr-exit-2-replay.html' | relative_url }}">Open the simulator in a full page</a>.</figcaption></figure>
<script>
(() => {
  const frame = document.getElementById('fdr-replay-frame');
  let observer;
  function fitReplay() {
    const content = frame.contentDocument?.querySelector('main');
    if (!content) return;
    observer?.disconnect();
    const resize = () => { frame.style.height = Math.ceil(content.getBoundingClientRect().height) + 'px'; };
    observer = new ResizeObserver(resize);
    observer.observe(content);
    resize();
  }
  frame.addEventListener('load', fitReplay);
  fitReplay();
})();
</script></section>
<p class="fdr-source"><em>Updated November 1, 2026. The 2025 rule comparison uses the reference model. The Exit 2 simulator uses the weighted model’s 2026 replay.</em></p>
</div>
