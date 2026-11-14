---
layout: project
permalink: /projects/2026-nyc-traffic-analysis/original-study/
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

**Earlier study · September 2026.** This write-up covers two northbound FDR segments, before I narrowed the project to segment 217. [Read the later segment 217 results and Exit 2 simulator]({{ '/projects/2026-nyc-traffic-analysis/' | relative_url }}).

During my commute to work, I sometimes reached a congested road before realizing I needed another route. By then, there were few options left. I started looking at NYC traffic data to see whether I could get that information earlier.

The project began with sensor reliability across the five boroughs, then narrowed to two segments of FDR Drive. While a road was still flowing, could its recent speeds tell me that congestion would begin in the next 30 minutes?

## What counts as a congestion event? {#fdr-2}

An event starts when a road segment's speed drops below the threshold set for that segment. If it recovers above the threshold and drops again, I count another event, even when the recovery is brief. That makes the definition reproducible, though a commuter might experience both events as one continuing disruption.

Road speeds let me test advance warnings. They did not tell me whether someone could still change routes. I had no individual journeys, alternative-route times, or arrival deadlines to test that decision.

### The two road segments {#fdr-study-area}

I chose northbound FDR segments 217 and 215 because they frequently experience morning congestion. Segment 217 is labeled Catherine Slip–25th Street, and segment 215 is labeled 25th–63rd Street. These IDs identify road segments, not vehicles.

[Open the study-area map]({{ '/assets/interactive/nyc-fdr-study-map.html' | relative_url }}). It shows the location of the study, not live warnings. The recorded coordinates are incomplete, so the highlighted lines cover only parts of each segment. I left the missing sections unfilled.

The [earlier citywide map]({{ '/assets/interactive/congestion_error_map.html' | relative_url }}) shows sensor reliability and evening peak congestion from April–July 2024. Those historical layers are separate from the FDR model's predictions, and the cached road geometry may be incomplete. Both maps need an internet connection for their map libraries and background tiles.

## How much notice did I count? {#fdr-3}

I counted warnings issued 5–30 minutes before congestion began. For the later comparisons, I also counted the subset that arrived at least 10 minutes ahead. I chose that cutoff for the study. I did not measure whether 10 minutes would be enough for a commuter to change plans.

Before comparing models, I needed to check the timestamps and missing observations. I also needed to decide when another alert was useful and when it was just a repeat.

## Checking the data and the warning rules {#fdr-4}

### Missing records changed the evaluation

The initial dataset contained 4,233,169 NYC DOT sensor records from April through July 2024. I measured reliability as the share of received records without the error status `-101`. That share was 93.4% in Queens but only 56.3% in Manhattan. It describes the records I received, not readings that never arrived.

<figure>
  <img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Records without the error flag by borough: Queens 93.4%, Bronx 83.7%, Staten Island 65.1%, Brooklyn 62.1%, and Manhattan 56.3%." loading="lazy">
  <figcaption>Share of received records without the error flag, by borough.</figcaption>
</figure>

In the early analysis, I interpolated gaps in the sensor data. For the warning model, I limited that to small gaps in past inputs, using only observations available at prediction time. Other missing-input treatments improved some measures, but none consistently improved warnings while staying within the chosen alert limits. I kept the limited interpolation.

I also found a timestamp problem. Moving six rows forward only means 30 minutes ahead when every reading is exactly five minutes apart. A missing record could move the target further into the future. I changed the evaluation to match actual timestamps. A prediction made at 9:00 needed the observation at 9:30.

Missing past inputs could prevent a prediction altogether. Missing future observations meant an alert could be issued but its outcome could not be scored. I tracked those cases separately and never filled future gaps to create evaluation targets.

### Predicting the start of congestion

My first approach predicted speed exactly 30 minutes ahead. A road could slow down after 10 minutes and recover before minute 30, so an accurate final-speed prediction could still miss the event.

I compared predictions at 5, 10, 15, 20, 25, and 30 minutes ahead, then tried predicting whether congestion would start anywhere in that window. For this onset task, I tested logistic regression, random forest, and XGBoost with four settings each. A random forest became the reference model.

I trained the onset models on 2023 data and used 2024 to compare candidates and save the selection before evaluating 2025. I had already inspected 2025 in earlier work, so this was an exploratory evaluation. The target, model, and training setup changed together. The results cannot isolate the effect of changing the target alone.

### Deciding when to issue another alert

Every five minutes, the model checks a flowing road if the required past observations are available. The random forest averages predictions from 300 decision trees to score congestion starting in the next 30 minutes. A higher score indicates a stronger prediction, but a score of 0.7 is not necessarily a calibrated 70% probability.

A score of at least 0.5 creates an alert candidate. I first fixed that threshold to compare models under the same conditions. Lower thresholds later caught more events but produced more false alarms and more alerts overall. I kept 0.5 under the study's alert limits, without claiming it was optimal.

The alert rule then determines whether to issue the candidate or suppress a repeated warning. The 30-minute prediction window and the 30-minute suppression period have different jobs. One looks ahead for congestion, while the other starts after an alert and limits repeats.

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-warning-workflow.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-warning-workflow.svg' | relative_url }}" alt="Check current and past observations, calculate a congestion score, apply the repeat-alert rule, and record an alert. Missing inputs, a low score, or suppression can stop the process." loading="lazy">
  </a>
  <figcaption>Future observations are used to evaluate an alert later. They are never used to decide whether to issue it. Select the diagram to enlarge it.</figcaption>
</figure>

Keeping the model and threshold fixed, I tested a different repeat-alert rule. After observing congestion, it would wait for 5, 10, or 15 minutes of continuous recovery before allowing another warning. Five minutes of recovery requires two flowing observations five minutes apart. A missing observation breaks the sequence.

This rule could allow another alert before 30 minutes had passed if congestion cleared quickly. It could also block alerts for longer while waiting for recovery. If no congestion had been observed after an alert, the rule required both 30 minutes to pass and continuous recovery to be confirmed.

## What changed in the results {#fdr-5}

The following comparisons use the same 1,045 congestion events in 2025 across the two FDR segments. A detection means an issued alert matched an upcoming event. I did not count one alert as a success for multiple events. Missing observations could leave an alert's outcome unknown.

The result charts use saved experiment data. The recovery timeline further below is a hypothetical example. Each figure links to a larger image.

### Earlier detection came with more alerts

Predicting one future speed caught 78 events, or 7.5%. Predicting six future speeds caught 120, or 11.5%. The direct onset model caught 398, or 38.1%. Of those 398 detections, 242 gave at least 10 minutes' notice, representing 23.2% of all 1,045 events.

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-prediction-progress.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-prediction-progress.svg' | relative_url }}" alt="Advance detection out of 1,045 events: one future speed, 78 or 7.5%. Six future speeds, 120 or 11.5%. Direct onset prediction, 398 or 38.1%." loading="lazy">
  </a>
  <figcaption>Figure 1. The onset model caught more events, but the model and training setup changed along with the prediction target.</figcaption>
</figure>

The reference model issued 829 alerts. Of these, 398 matched events, 190 were false alarms, and 241 could not be scored. The false-alert rate was 32.3% among the 588 scoreable alerts. The 241 unknown outcomes are outside that denominator.

A separate class-weighting experiment caught 558 events, or 53.4%, but issued 1,520 alerts. It produced 498 false alarms and 464 unknown outcomes. Compared with the reference, that meant 160 more detections, 308 more false alarms, and 223 more unknown outcomes. I did not adopt it under the experiment's alert-burden limits. Those were conservative comparison rules, not measured user preferences.

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.svg' | relative_url }}" alt="Reference model: 398 matched alerts, 190 false, 241 unknown, 829 total. Weighted model: 558 matched, 498 false, 464 unknown, 1,520 total." loading="lazy">
  </a>
  <figcaption>Figure 2. Class weighting increased detections and the number of alerts that were false or could not be evaluated.</figcaption>
</figure>

Adding speed-change and neighboring-road inputs also failed to produce a replacement that met the selection requirements and improved timely detection in the 2025 evaluation.

### Waiting for recovery blocked useful warnings

| Alert rule | Events detected 5–30 minutes ahead | At least 10 minutes ahead | Alerts issued |
| --- | --- | --- | --- |
| Existing 30-minute suppression | 398 · 38.1% | 242 · 23.2% | 829 |
| Confirm 5 minutes of recovery | 362 · 34.6% | 215 · 20.6% | 754 |
| Confirm 10 minutes of recovery | 289 · 27.7% | 180 · 17.2% | 644 |
| Confirm 15 minutes of recovery | 257 · 24.6% | 168 · 16.1% | 570 |

The percentages use all 1,045 events. Detections at least 10 minutes ahead are a subset of the 5–30-minute detections.

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-recovery-rules.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-recovery-rules.svg' | relative_url }}" alt="Overall and at least 10-minute detection rates: existing rule 38.1% and 23.2%. Five-minute recovery 34.6% and 20.6%. Ten-minute recovery 27.7% and 17.2%. Fifteen-minute recovery 24.6% and 16.1%." loading="lazy">
  </a>
  <figcaption>Figure 3. Longer recovery requirements reduced both overall detection and warnings with at least 10 minutes' notice. The green bars are a subset of the blue bars.</figcaption>
</figure>

The five-minute rule found 44 events that the existing rule missed, but lost 80 it had caught. In all 80 losses, the model still produced a candidate at the earlier successful warning time. The new rule blocked it while waiting for another flowing observation. In 66 cases, congestion returned five minutes later, before recovery could be confirmed.

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-recovery-timeline.svg' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-recovery-timeline.svg' | relative_url }}" alt="Hypothetical example: an alert at 08:00, congestion at 08:10, brief flow at 08:40, and renewed congestion at 08:45. The existing rule can warn at 08:40, while the recovery rule waits and misses the opportunity." loading="lazy">
  </a>
  <figcaption>A hypothetical timeline, not a recorded event. Waiting for a second flowing observation can lose the chance to warn before congestion returns. Columns are not to time scale.</figcaption>
</figure>

I kept the existing 30-minute rule. The recovery rule sent fewer alerts, but the event-level check showed which useful warnings it removed.

### Where the missed events came from

The reference model missed 647 events. I grouped them by what happened in the warning process:

| What happened | Missed events | Share of the 647 misses |
| --- | --- | --- |
| Required inputs unavailable | 158 | 24.4% |
| Model ran but produced no candidate | 275 | 42.5% |
| Candidates existed without a scored detection | 214 | 33.1% |

<figure>
  <a href="{{ '/assets/img/projects/nyc-fdr-missed-events.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-missed-events.svg' | relative_url }}" alt="Of 647 missed events, 158 had unavailable inputs, 275 had no candidate, and 214 had candidates without a scored detection." loading="lazy">
  </a>
  <figcaption>Figure 4. Input availability and candidate generation account for 433 misses that changing the delivery rule alone cannot recover.</figcaption>
</figure>

The final group includes suppression, missing observations, and alert-to-event matching. It cannot all be attributed to the repeat-alert rule.

With the reference model and threshold fixed, only 558 events had any scoreable candidate opportunity. Even that is an optimistic ceiling because it ignores suppression and competition between events for alerts. Delivery-rule changes alone could not reach 70% detection. This 558-event count is a diagnostic for the fixed reference model. It happens to equal the detections in the separate class-weighting experiment, but the two counts describe different things.

## What this evaluation leaves open {#fdr-6}

The results cover two FDR segments and a period I inspected repeatedly. They do not establish performance on other roads, across NYC, or on rail services. I would need a new evaluation period to check whether a change holds up.

The event definition also matters. A brief recovery splits congestion into two events here, though a driver might never experience a meaningful break. I have not tested whether a different definition would better match that experience.

This was an offline evaluation. It did not measure actual data delivery delays, route changes, notification fatigue, travel-time savings, or reduced lateness. Passing code tests cannot establish those outcomes, and missing future observations still leave some issued alerts unjudged.

## Where I stopped {#fdr-7}

I closed this phase with the random forest, score threshold of 0.5, and 30-minute suppression rule as the reference configuration. The recovery experiments explained why I was keeping that rule. Continuing to tune it on the same period would not answer whether it works on new data.

A next evaluation would freeze the model and alert rule, check when observations actually become available, and run on a new period without sending live notifications. The missing-input cases also need attention. I would check which observations prevent predictions and whether a model needing fewer inputs can recover useful opportunities.

One rule remains untested: allow early release after confirmed recovery, but release after 30 minutes even without it. Earlier warnings could alter later suppression, so the effect needs a full replay rather than an assumption that it would improve detection.

## Returning to the commuting question {#fdr-8}

I started with a route choice I wished I had made earlier. Working through the missed events showed how much sits between a prediction and a warning someone can use. Missing inputs can prevent a prediction, and the delivery rule can block a candidate even after the model identifies the risk.

To test the original idea, I would need journey and alternative-route data to identify the last point where a warning could still help. I would also need to learn what alert burden users accept. Those questions remain open.

I am interested in whether earlier passenger-facing information about rail disruptions could support similar decisions. That would need a separate dataset, a definition of disruption, and a review of the information passengers already receive.

*Experiments completed through September 29, 2026. The recovery-rule experiment and loss diagnosis are recorded under `20260929T054130Z` in the project repository.*
