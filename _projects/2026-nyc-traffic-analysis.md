---
title: "NYC Traffic Analysis: Can Congestion Be Warned About in Advance?"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "A commuting experience led me to test congestion warnings on NYC’s FDR Drive. I corrected time and missing-data issues, compared prediction and alert rules, and closed this phase with clear limits and next steps."
thumbnail: /assets/img/projects/nyc-fdr-prediction-progress.png
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
#fdr-study .fdr-badge{display:inline-block;border-radius:5px;padding:4px 9px;font-size:12px;line-height:1.5;font-weight:700;background:#e0f0e9;color:#226357}
#fdr-study .fdr-badge.amber{background:#fff0d9;color:#805011}
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
#fdr-study .fdr-callout{padding:18px 20px;border-left:4px solid var(--teal);background:#eef7f5;border-radius:0 8px 8px 0;margin:20px 0}
#fdr-study .fdr-callout.amber{border-color:#ad6a18;background:#fff5e5}
#fdr-study .fdr-flow{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:10px;margin:20px 0;list-style:none;padding:0}
#fdr-study .fdr-flow li{padding:16px;background:#f1f6f8;border-radius:8px;font-size:13px;margin:0}
#fdr-study .fdr-flow span{display:block;font-size:11px;font-weight:bold;color:var(--teal);margin-bottom:8px}
#fdr-study .fdr-flow strong{display:block;font-size:16px;margin-bottom:8px}
#fdr-study details{border-top:1px solid var(--line);margin:18px 0 0;padding-top:14px}
#fdr-study summary{cursor:pointer;font-weight:650;color:var(--teal);padding:5px 0}
#fdr-study .fdr-table{overflow:auto}
#fdr-study table{border-collapse:collapse;width:100%;font-size:13px;margin:14px 0}
#fdr-study th,#fdr-study td{text-align:left;padding:12px;border-bottom:1px solid var(--line);vertical-align:top}
#fdr-study th{background:#f2f6f8;color:var(--sub)}
#fdr-study li{margin:10px 0}
#fdr-study .fdr-roadmap{counter-reset:next;list-style:none;padding:0}
#fdr-study .fdr-roadmap li{counter-increment:next;padding:16px 18px 16px 54px;background:#f3f7f9;border-radius:8px;position:relative}
#fdr-study .fdr-roadmap li:before{content:counter(next);position:absolute;left:18px;top:17px;color:var(--teal);font-weight:750}
#fdr-study .fdr-source{font-size:12px;color:var(--sub);overflow-wrap:anywhere}
#fdr-study .fdr-reflection{border-left:4px solid var(--teal)}
@media(max-width:650px){#fdr-study{padding:12px;margin-left:-8px;margin-right:-8px}#fdr-study .fdr-section,#fdr-study .fdr-hero{padding:20px 16px}#fdr-study .fdr-metrics{grid-template-columns:1fr}#fdr-study .fdr-metric{padding:16px}#fdr-study .fdr-metric strong{font-size:30px}#fdr-study .fdr-flow{grid-template-columns:1fr 1fr}#fdr-study h2,#fdr-study .fdr-hero h2{font-size:24px}#fdr-study figcaption{padding:12px;font-size:12px}}
@media(prefers-color-scheme:dark){#fdr-study{background:#14222b;--ink:#e2eff3;--sub:#b6c9d2;--line:#34505e;--teal:#8ad4cc}#fdr-study .fdr-section,#fdr-study .fdr-metric,#fdr-study .fdr-nav a{background:#1b303b}#fdr-study a{color:#8ad4cc}#fdr-study .fdr-flow li,#fdr-study .fdr-roadmap li,#fdr-study th{background:#243f4b}#fdr-study figcaption{background:#1b303b}#fdr-study .fdr-callout{background:#203d3b}#fdr-study .fdr-callout.amber{background:#493a25}#fdr-study .fdr-nav a:hover{background:#284a50}}
</style>
<div id="fdr-study">
<header class="fdr-hero"><span class="fdr-eyebrow">Mobility data · prediction · operational decisions</span><h2>Could an earlier warning have changed my commute?</h2><span class="fdr-badge">Research phase complete</span> <span class="fdr-badge amber">Offline evaluation</span>
<p>During my commute to work, I found that once I reached a congested road segment, my options for taking another route were limited. I wanted a warning before reaching those segments, while I still had time to choose a different route. That experience led me to ask: could congestion be predicted early enough to help me make that choice?</p>
<p>I started looking at NYC traffic data to see whether an early warning system was something I could actually build. The project began with sensor reliability across the five boroughs, then narrowed to two northbound segments of the FDR Drive. My question became: <strong>when a road is still flowing, can I warn that congestion will start within the next 30 minutes?</strong></p>

<p><strong>I'm wrapping up this phase here.</strong> The latest alert rule reduced useful warnings, so I kept the earlier approach as the comparison baseline. The results below show what changed, why some alternatives were rejected, and what I would investigate next.</p></header>
<div class="fdr-metrics" aria-label="Reference results on 1,045 evaluated congestion events in 2025">
<div class="fdr-metric"><span class="fdr-eyebrow">Advance detection</span><strong>38.1%</strong><small>398 of 1,045 events<br>Warned 5–30 minutes ahead</small></div>
<div class="fdr-metric"><span class="fdr-eyebrow">More time to act</span><strong>23.2%</strong><small>242 of the same 1,045 events<br>Warned at least 10 minutes ahead</small></div>
<div class="fdr-metric"><span class="fdr-eyebrow">Unknown outcomes</span><strong>241 / 829</strong><small>Issued alerts that could not be scored<br>because observations were missing</small></div></div>
<nav class="fdr-nav" aria-label="Case study sections"><a href="#fdr-2">The question</a><a href="#fdr-study-area">Study area</a><a href="#fdr-4">How it works</a><a href="#fdr-5">Results</a><a href="#fdr-6">Limitations</a><a href="#fdr-7">Next steps</a></nav>
<section id="fdr-2" class="fdr-section" aria-labelledby="fdr-title-2"><span class="fdr-eyebrow">01 / Case study</span><h2 id="fdr-title-2">Problem</h2><p>Knowing that a road is already congested is different from knowing early enough to reconsider a route. I wanted to study the second problem.</p>
<p>I could not test that whole commuting decision with road speeds alone. I did not have individual journeys, alternative routes, or arrival deadlines. Instead, I tested one part of a possible system: whether observed speeds could give advance notice of congestion on a road segment.</p>
<p>For this test, a congestion event starts when a road segment's speed drops below the thread set for that segment. If the speed recovers above the threadshold and then drops below it again, it counts as a new event, even if the recovery is brief. 'a congestion event starts when traffic on a road segment slows below a set speed thredshold.</p><h3 id="fdr-study-area">Where I focused the warning study</h3>
<p>I started by looking at sensor reliability across NYC, then narrowed the warning study to these two northbound FDR segments because they frequently expeirence congestion during the morning commute.  The IDs below identify road segments, not individual vehicles.</p>
<figure><iframe src="{{ '/assets/interactive/nyc-fdr-study-map.html' | relative_url }}" title="Study area: partial recorded coordinates for FDR northbound road segments 217 and 215" loading="lazy" style="display:block;width:100%;height:430px;border:0;"></iframe><figcaption><strong>Study location, not a live warning map.</strong> Road 217 is labeled Catherine Slip–25th St, and road 215 is labeled 25th–63rd St. The available road coordinates are incomplete, so the highlighted lines show only parts of these segments. Missing sections have not been filled in. An internet connection is required to display the background map. <a href="{{ '/assets/interactive/nyc-fdr-study-map.html' | relative_url }}">Open the study map in a full page</a>.</figcaption></figure>
<details><summary>Explore the initial citywide analysis</summary>
<p>This earlier map shows sensor reliability and PM-peak congestion across the monitored NYC network. It provides the starting context for the project its historical layers are not predictions from the final FDR warning model. Cached road geometry may be incomplete.</p>
<iframe src="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}" title="Earlier exploratory map of NYC sensor reliability and PM-peak congestion" loading="lazy" style="display:block;width:100%;height:480px;border:1px solid #dbe5ea;border-radius:8px;"></iframe>
<p class="fdr-source"><a href="{{ '/assets/interactive/congestion_error_map.html' | relative_url }}">Open the original citywide map in a full page</a>. Initial analysis: April–July 2024. Interactive map libraries and base-map tiles require internet access.</p></details>
</section>
<section id="fdr-3" class="fdr-section" aria-labelledby="fdr-title-3"><span class="fdr-eyebrow">02 / Case study</span><h2 id="fdr-title-3">Goals</h2><p>I wanted to answer three questions:</p>
<ul><li>Could I trust the timestamps and missing-data handling enough to evaluate a forecast fairly?</li><li>Could I identify new congestion while the road was still flowing, with time to act?</li><li>Could I limit repeated alerts without blocking useful warnings?</li></ul>
<p>I evaluated warnings issued 5–30 minutes before congestion began, then focused the follow-up experiments on warnings that gave at least 10 minutes’ notice. I chose 10 minutes for this study, but did not test whether that would give commuters enough time to change their plans.</p></section>
<section id="fdr-4" class="fdr-section" aria-labelledby="fdr-title-4"><span class="fdr-eyebrow">03 / Case study</span><h2 id="fdr-title-4">Process</h2><h3>First, I checked the data</h3>
<p>The initial analysis covered 4,233,169 NYC DOT sensor records from April through July 2024. Reliability, measured as the share of records without the error status <code>-101</code>, varied considerably: 93.4% in Queens versus 56.3% in Manhattan. That made data availability part of the prediction problem from the start.</p>
<img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Valid sensor record share by borough: Queens 93.4%, Bronx 83.7%, Staten Island 65.1%, Brooklyn 62.1%, and Manhattan 56.3%." style="max-width:100%;" loading="lazy">
<p>In the early analysis, I used interpolation to fill gaps in the sensor data. For the later warning system, I only filled small gaps in the past data used for prediction, using observations already available at the time.</p>
<p>I also found a mismatch in timestamps. Moving six records forward only means 30 minutes ahead if every record is exactly five minutes apart. When records are missing, it can point further into the future. I changed the evaluation to match actual timestamps. A 9:00 prediction needed a 9:30 observation, not whichever value happened to be six rows later.</p>
<p>I then tested other ways to handle missing inputs. Some improved certain measures, but none consistently produced better warnings while keeping the number of alerts within the chosen limits. I therefore kept the limited use of interpolation for past inputs in the final system.</p>
<p>Missing inputs and missing future observations had different effects. If the required past data were still unavailable, the model could not make a prediction. If a future observation was missing, the system could still issue a warning, but I could not evaluate it. I kept these cases separate to show when the system could make predictions and when those predictions could be evaluated. Missing future observations were never filled in to create evaluation targets.</p>
<h3>Then, I changed the prediction question</h3>
<p>For the FDR warning study, my first approach predicted speed exactly 30 minutes ahead. But a road could become congested after 10 minutes and recover before the 30-minute mark. A good prediction of the final speed could still miss the event I cared about.</p>
<p>I first compared speed predictions at six points in time: 5, 10, 15, 20, 25, and 30 minutes ahead. I then trained models to predict whether congestion would begin at any point within the next 30 minutes. I tested logistic regression, random forest, and XGBoost with four settings each. A random forest became the reference model.</p>
<p>For the onset experiments, I trained on 2023 data and used 2024 to compare candidates and save the selection before evaluating 2025. However, I had already inspected 2025 in earlier work, so these are exploratory results rather than a final untouched test. The target, model, and training setup changed together; I cannot attribute the improvement to the target alone.</p>
<h3>Finally, I separated prediction from alert delivery</h3>
<h3>Finally, I separated prediction from alert delivery</h3>

<p>Every five minutes, the model checks a currently flowing road if the required inputs are available. The Random Forest averages the predictions from its 300 decision trees to produce a score for congestion starting within the next 30 minutes. Higher scores indicate a stronger prediction of congestion, but a score of 0.7 does not necessarily mean a 70% chance of congestion.</p>

<p>I initially fixed the threshold at 0.5 to compare models under the same conditions. I later tested lower thresholds, which detected more events early but also produced more false alarms and more alerts overall. I kept 0.5 because of this trade-off under the study's alert limits, rather than because it was proven optimal.</p>

<p>A score of at least 0.5 creates an alert candidate. The alert rules then decide whether to send the warning or hold it back to avoid repeated alerts.</p>
<ol class="fdr-flow"><li><span>01 · OBSERVE</span><strong>Check the road</strong>Current flow and required past records.</li><li><span>02 · PREDICT</span><strong>Score the risk</strong>Will congestion begin within 30 minutes?</li><li><span>03 · FILTER</span><strong>Check the rule</strong>A high score can still be suppressed.</li><li><span>04 · WARN</span><strong>Record an alert</strong>Issue only when both conditions pass.</li></ol><details><summary>See the full workflow and where warnings can stop</summary><figure><a href="{{ '/assets/img/projects/nyc-fdr-warning-workflow.png' | relative_url }}"><img src="{{ '/assets/img/projects/nyc-fdr-warning-workflow.svg' | relative_url }}" alt="Four stages: check current and past observations, calculate risk, apply the repeat-alert rule, and record an alert. Missing inputs, low scores, or suppression can stop the process." loading="lazy"></a><figcaption>How it works. Prediction and delivery are separate decisions. Future observations are used later for evaluation, never to decide whether to issue an alert.</figcaption></figure></details><p class="fdr-callout">There are two separate clocks: the <strong>30-minute prediction window</strong> looks ahead for congestion, while the <strong>30-minute suppression period</strong> limits repeat alerts after a warning. They happen to have the same length here, but they serve different purposes.</p>
<p>I held the model and score threshold fixed while testing a different rule: after observing congestion, wait for 5, 10, or 15 minutes of continuous recovery before allowing another warning. Five minutes of recovery requires two flowing observations five minutes apart. Missing observations break that sequence.</p>
<p>This could release the restriction earlier than 30 minutes if congestion cleared quickly, but it could also keep alerts blocked longer while waiting for recovery. If no congestion had been observed after an alert, the new rule required both 30 minutes to pass and continuous recovery to be confirmed before releasing the restriction.</p></section>
<section id="fdr-5" class="fdr-section" aria-labelledby="fdr-title-5"><span class="fdr-eyebrow">04 / Case study</span><h2 id="fdr-title-5">Results and Decisions</h2><p><em>The four result charts use saved experiment data. The separate timeline is a hypothetical explanation. Select any image to enlarge it.</em></p>
<p>All detection counts below use the same 1,045 evaluated congestion events in 2025. A detection means an issued alert was matched to an upcoming event. One alert was not counted as a success for multiple events; missing observations could leave its outcome unknown.</p>
<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-prediction-progress.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-prediction-progress.svg' | relative_url }}" alt="Advance detection on 1,045 events: one future speed, 78 events (7.5%); six future speeds, 120 (11.5%); direct onset prediction, 398 (38.1%)." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 1. Predicting congestion onset was more useful for this question. The model and training setup also changed, so this comparison does not isolate the effect of changing the target.</figcaption>
</figure>
<p>The direct approach was more useful for this question, but better detection came with more alerts. A later class-weighting experiment detected 558 events, or 53.4%, while increasing false alerts from 190 to 498 and total alerts from 829 to 1,520. I did not adopt it under the experiment's alert-burden constraints. Those constraints were conservative comparison rules, not measured user preferences.</p>
<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.svg' | relative_url }}" alt="Reference alerts: 398 matched, 190 false, 241 unknown, 829 total. Weighted model: 558 matched, 498 false, 464 unknown, 1,520 total." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 2. The weighted model found 160 more events, but added 308 false alerts and 223 alerts with unknown outcomes. I kept the reference configuration under the study’s burden constraints.</figcaption>
</figure>
<p>Adding speed-change and neighboring-road inputs also did not produce a replacement that met the selection requirements and improved timely detection in the 2025 evaluation.</p>
<h3>Waiting for recovery reduced useful warnings</h3>
<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-recovery-rules.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-recovery-rules.svg' | relative_url }}" alt="Overall and at least 10-minute detection: existing rule 38.1% and 23.2%; recovery 5 minutes 34.6% and 20.6%; recovery 10 minutes 27.7% and 17.2%; recovery 15 minutes 24.6% and 16.1%." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 3. Waiting longer for recovery reduced both overall detection and warnings with at least 10 minutes to act. The green bars are a subset of the blue bars, not additional detections.</figcaption>
</figure>
<details><summary>View exact event and alert counts</summary><div class="fdr-table"><table><thead><tr><th scope="col">Alert rule</th><th scope="col">Advance detection</th><th scope="col">At least 10 minutes ahead</th><th scope="col">Total alerts</th></tr></thead><tbody><tr><td>Existing 30-minute suppression</td><td>398 · 38.1%</td><td>242 · 23.2%</td><td>829</td></tr><tr><td>Confirm 5 minutes of recovery</td><td>362 · 34.6%</td><td>215 · 20.6%</td><td>754</td></tr><tr><td>Confirm 10 minutes of recovery</td><td>289 · 27.7%</td><td>180 · 17.2%</td><td>644</td></tr><tr><td>Confirm 15 minutes of recovery</td><td>257 · 24.6%</td><td>168 · 16.1%</td><td>570</td></tr></tbody></table></div></details>
<p>The five-minute rule found 44 events that the existing rule missed, but lost 80 that it previously caught. In all 80 losses, the model still produced a candidate at the earlier successful alert time. The new rule blocked it while waiting for another flowing observation. In 66 cases, congestion returned five minutes later—before recovery could be confirmed.</p>
<figure><a href="{{ '/assets/img/projects/nyc-fdr-recovery-timeline.png' | relative_url }}"><img src="{{ '/assets/img/projects/nyc-fdr-recovery-timeline.svg' | relative_url }}" alt="Hypothetical sequence: an alert at 08:00, congestion at 08:10, brief flow at 08:40, and renewed congestion at 08:45. The existing rule can allow a warning at 08:40, while the recovery rule waits and misses that opportunity." loading="lazy"></a><figcaption>Illustrative timeline, not a recorded event. At 08:40, the existing rule can issue a new high-score candidate. Waiting for a second flowing observation leaves no warning before congestion returns. Columns are not to time scale.</figcaption></figure>
<p class="fdr-callout">That helped explain why fewer alerts were not automatically an improvement. The model had identified risk, but the delivery rule stopped the warning. <strong>I kept the existing 30-minute rule.</strong></p>
<h3>The remaining failures were not all model failures</h3>
<p>The reference approach missed 647 events. I separated them by where the warning process broke down:</p>
<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-missed-events.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-missed-events.svg' | relative_url }}" alt="Of 647 missed events: required inputs unavailable, 158 (24.4%); model ran but no candidate, 275 (42.5%); candidates without scored detection, 214 (33.1%)." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 4. Percentages are shares of the 647 missed events. Input availability and candidate generation account for 433 misses that changing the alert-delivery rule alone cannot recover.</figcaption>
</figure>
<p>The last group includes suppression, missing observations, and alert-to-event matching; it cannot all be blamed on the repeat-alert rule. With the current model and score threshold fixed, only 558 events had any scoreable candidate opportunity. Even that is an optimistic ceiling that ignores suppression and competition between events for alerts. Changing delivery rules alone cannot reach 70% detection. This 558-event opportunity count is a diagnostic for the fixed reference model, not the result of the separate class-weighting experiment above, which happened to detect the same number of events.</p></section>
<section id="fdr-6" class="fdr-section" aria-labelledby="fdr-title-6"><span class="fdr-eyebrow">05 / Case study</span><h2 id="fdr-title-6">Limitations</h2><ul><li><strong>The study covers two FDR segments.</strong> It does not establish performance across NYC, other roads, or rail services.</li><li><strong>The evaluation period has been inspected repeatedly.</strong> A new period is needed before claiming that improvements generalize.</li><li><strong>Some alerts cannot be judged.</strong> Of the reference system's 829 alerts, 190 were false alerts and 241 had insufficient observations to score. The 32.3% false-alert rate applies only to the 588 scoreable alerts.</li><li><strong>An event is not the same as a commuter's experience.</strong> Brief flowing intervals split congestion into separate events here. Whether those should count as one continuing disruption needs further study.</li><li><strong>This was an offline evaluation.</strong> Actual data delivery delays, route changes, notification fatigue, travel-time savings, and reduced lateness were not measured. Passing code tests does not establish those outcomes.</li></ul></section>
<section id="fdr-7" class="fdr-section" aria-labelledby="fdr-title-7"><span class="fdr-eyebrow">06 / Case study</span><h2 id="fdr-title-7">Where I&#x27;m Stopping, and What Comes Next</h2><p class="fdr-callout amber">I'm keeping the random forest, score threshold of 0.5, and 30-minute suppression as the reference configuration and closing this phase of the project. I have enough evidence to explain why the recovery rules were not adopted. Continuing to change rules on the same data would not resolve the bigger question of whether this works in a new period or helps a commuter.</p>
<p>If I return to the project, I would prioritize:</p>
<ol class="fdr-roadmap"><li><strong>A new evaluation period.</strong> Freeze the model and alert rule, check when observations actually become available, and evaluate without sending live notifications.</li><li><strong>The missing-input cases.</strong> Find which required observations prevent predictions, then test whether a smaller-input model can recover useful opportunities.</li><li><strong>One bounded alert-rule experiment.</strong> Allow early release after confirmed recovery, but release after 30 minutes even without it. This has not been tested, and earlier alerts could change later suppression, so improvement is not guaranteed.</li><li><strong>The original commuting question.</strong> Identify when a warning would still allow a different route and what alert burden users would accept. Alternative-route and journey data would be needed to measure whether warnings actually help.</li></ol>
<p>I would also like to explore whether earlier passenger-facing rail disruption information could support similar travel decisions. That would require a separate dataset, a definition of disruption, and a review of what information passengers already receive.</p></section>
<section id="fdr-8" class="fdr-section fdr-reflection" aria-labelledby="fdr-title-8"><span class="fdr-eyebrow">07 / Case study</span><h2 id="fdr-title-8">Reflection</h2><p>I started because I wanted information that might have helped me avoid a bad route choice. I expected most of the work to be about predicting traffic. In practice, a lot of it came down to deciding what counted as a useful warning, checking that time and missing data were handled correctly, and understanding why a prediction did not turn into an alert.</p>
<p>The most useful result was not always a higher detection rate. Finding that a reasonable-looking recovery rule blocked warnings gave me a clearer reason to keep the simpler approach. I have not built a system that can promise to prevent late arrivals, but I now have a more concrete understanding of what would need to work before making that claim.</p>
</section>
<p class="fdr-source"><em>This page summarizes experiments completed through September 29, 2026. The latest recovery-rule experiment and loss diagnosis are recorded under <code>20260929T054130Z</code> in the project repository.</em></p>
</div>
