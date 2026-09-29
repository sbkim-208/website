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

### 1. Overview

*Status: This research phase is complete. The warning system has been evaluated offline and has not been deployed.*

When I was commuting to work, I chose a route without realizing how bad the traffic ahead would become and ended up late. That experience stayed with me. If I had known earlier, could I have made a different choice?

I started looking at NYC traffic data to see whether an early warning system was something I could actually build. The project began with sensor reliability across the five boroughs, then narrowed to two northbound segments of the FDR Drive. My question became: **when a road is still flowing, can I warn that congestion will start within the next 30 minutes?**

The reference model detected 398 of 1,045 evaluated congestion events in advance, or 38.1%. Only 23.2% were detected at least 10 minutes ahead. That was enough to make the idea worth investigating, but it did not establish that the system would help someone avoid being late.

**I'm wrapping up this phase here.** The latest alert rule reduced useful warnings, so I kept the earlier approach as the comparison baseline. The four result figures below show how the prediction question changed, why higher detection was not enough, and where the remaining failures occurred.

### 2. Problem

Knowing that a road is already congested is different from knowing early enough to reconsider a route. I wanted to study the second problem.

I could not test that whole commuting decision with road speeds alone. I did not have individual journeys, alternative routes, or arrival deadlines. Instead, I tested one part of a possible system: whether observed speeds could give advance notice of congestion on a road segment.

For this study, a congestion event begins when a segment changes from flowing to below its congestion-speed threshold. It is a road-state transition, not a record of someone arriving late. A brief recovery followed by another slowdown counts as a new event under this definition.

### 3. Goals

I wanted to answer three questions:

- Could I trust the timestamps and missing-data handling enough to evaluate a forecast fairly?
- Could I identify new congestion while the road was still flowing, with time to act?
- Could I limit repeated alerts without blocking useful warnings?

I evaluated both warnings issued 5–30 minutes before an event and the more demanding case of at least 10 minutes' notice. The latter became the main measure in the follow-up experiments. Ten minutes was a research setting; I have not checked whether it is enough for a real commuter to change plans.

### 4. Process

#### First, I checked the data

The initial analysis covered 4,233,169 NYC DOT sensor records from April through July 2024. Reliability, measured as the share of records without the error status `-101`, varied considerably: 93.4% in Queens versus 56.3% in Manhattan. That made data availability part of the prediction problem from the start.

<img src="{{ '/assets/img/projects/nyc-traffic-borough-reliability.png' | relative_url }}" alt="Valid sensor record share by borough: Queens 93.4%, Bronx 83.7%, Staten Island 65.1%, Brooklyn 62.1%, and Manhattan 56.3%." style="max-width:100%;" loading="lazy">

The early exploration included interpolation, but the later warning evaluation treated missing future observations as unknown rather than filling them in to create answers.

I also found a time-alignment problem. Moving six records forward only means 30 minutes ahead if every record is exactly five minutes apart. When records are missing, it can point much further into the future. I changed the evaluation to match actual timestamps. A 9:00 prediction needed a 9:30 observation, not whichever value happened to be six rows later.

Missing inputs and missing answers needed separate treatment too. If the required past observations were unavailable, the model might not run. If future observations were unavailable, an alert might still be issued but could not be scored. Dropping both cases would hide how often the system could actually be used.

#### Then, I changed the prediction question

For the FDR warning study, my first approach predicted speed exactly 30 minutes ahead. But a road could become congested after 10 minutes and recover before the 30-minute mark. A good prediction of the final speed could still miss the event I cared about.

I compared forecasts at six future times, then trained models to predict whether congestion would start anywhere within the next 30 minutes. I tested logistic regression, random forest, and XGBoost with four settings each. A random forest became the reference model.

For the onset experiments, I trained on 2023 data and used 2024 to compare candidates and save the selection before evaluating 2025. However, I had already inspected 2025 in earlier work, so these are exploratory results rather than a final untouched test. The target, model, and training setup changed together; I cannot attribute the improvement to the target alone.

#### Finally, I separated prediction from alert delivery

Every five minutes, the model checks a currently flowing road if the required inputs are available. A score of at least 0.5 creates an alert candidate. This score is not a calibrated 50% probability. The alert rule then decides whether to issue it.

There are two separate clocks: the **30-minute prediction window** looks ahead for congestion, while the **30-minute suppression period** limits repeat alerts after a warning. They happen to have the same length here, but they serve different purposes.

I held the model and score threshold fixed while testing a different rule: after observing congestion, wait for 5, 10, or 15 minutes of continuous recovery before allowing another warning. Five minutes of recovery requires two flowing observations five minutes apart. Missing observations break that sequence.

This could release the restriction earlier than 30 minutes if congestion cleared quickly, but it could also keep alerts blocked longer while waiting for recovery. If no congestion had been observed after an alert, the new rule required both 30 minutes to pass and continuous recovery to be confirmed before releasing the restriction.

### 5. Results and Decisions

*The charts use saved experiment results, not illustrative data. Select a chart to open a larger image.*

All detection counts below use the same 1,045 evaluated congestion events in 2025. A detection means an issued alert was matched to an upcoming event. One alert was not counted as a success for multiple events; missing observations could leave its outcome unknown.


<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-prediction-progress.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-prediction-progress.svg' | relative_url }}" alt="Advance detection on 1,045 events: one future speed, 78 events (7.5%); six future speeds, 120 (11.5%); direct onset prediction, 398 (38.1%)." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 1. Predicting congestion onset was more useful for this question. The model and training setup also changed, so this comparison does not isolate the effect of changing the target.</figcaption>
</figure>


The direct approach was more useful for this question, but better detection came with more alerts. A later class-weighting experiment detected 558 events, or 53.4%, while increasing false alerts from 190 to 498 and total alerts from 829 to 1,520. I did not adopt it under the experiment's alert-burden constraints. Those constraints were conservative comparison rules, not measured user preferences.


<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-alert-tradeoff.svg' | relative_url }}" alt="Reference alerts: 398 matched, 190 false, 241 unknown, 829 total. Weighted model: 558 matched, 498 false, 464 unknown, 1,520 total." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 2. The weighted model found 160 more events, but added 308 false alerts and 223 alerts with unknown outcomes. I kept the reference configuration under the study’s burden constraints.</figcaption>
</figure>

Adding speed-change and neighboring-road inputs also did not produce a replacement that met the selection requirements and improved timely detection in the 2025 evaluation.

#### Waiting for recovery reduced useful warnings


<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-recovery-rules.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-recovery-rules.svg' | relative_url }}" alt="Overall and at least 10-minute detection: existing rule 38.1% and 23.2%; recovery 5 minutes 34.6% and 20.6%; recovery 10 minutes 27.7% and 17.2%; recovery 15 minutes 24.6% and 16.1%." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 3. Waiting longer for recovery reduced both overall detection and warnings with at least 10 minutes to act. The green bars are a subset of the blue bars, not additional detections.</figcaption>
</figure>

| Alert rule | Advance detection | At least 10 minutes ahead | Total alerts |
| --- | ---: | ---: | ---: |
| Existing 30-minute suppression | 398 · 38.1% | 242 · 23.2% | 829 |
| Confirm 5 minutes of recovery | 362 · 34.6% | 215 · 20.6% | 754 |
| Confirm 10 minutes of recovery | 289 · 27.7% | 180 · 17.2% | 644 |
| Confirm 15 minutes of recovery | 257 · 24.6% | 168 · 16.1% | 570 |

The five-minute rule found 44 events that the existing rule missed, but lost 80 that it previously caught. In all 80 losses, the model still produced a candidate at the earlier successful alert time. The new rule blocked it while waiting for another flowing observation. In 66 cases, congestion returned five minutes later—before recovery could be confirmed.

That helped explain why fewer alerts were not automatically an improvement. The model had identified risk, but the delivery rule stopped the warning. **I kept the existing 30-minute rule.**

#### The remaining failures were not all model failures

The reference approach missed 647 events. I separated them by where the warning process broke down:


<figure style="margin: 1.5rem 0;">
  <a href="{{ '/assets/img/projects/nyc-fdr-missed-events.png' | relative_url }}">
    <img src="{{ '/assets/img/projects/nyc-fdr-missed-events.svg' | relative_url }}" alt="Of 647 missed events: required inputs unavailable, 158 (24.4%); model ran but no candidate, 275 (42.5%); candidates without scored detection, 214 (33.1%)." style="display:block;width:100%;height:auto;" loading="lazy">
  </a>
  <figcaption>Figure 4. Percentages are shares of the 647 missed events. Input availability and candidate generation account for 433 misses that changing the alert-delivery rule alone cannot recover.</figcaption>
</figure>


The last group includes suppression, missing observations, and alert-to-event matching; it cannot all be blamed on the repeat-alert rule. With the current model and score threshold fixed, only 558 events had any scoreable candidate opportunity. Even that is an optimistic ceiling that ignores suppression and competition between events for alerts. Changing delivery rules alone cannot reach 70% detection. This 558-event opportunity count is a diagnostic for the fixed reference model, not the result of the separate class-weighting experiment above, which happened to detect the same number of events.

### 6. Limitations

- **The study covers two FDR segments.** It does not establish performance across NYC, other roads, or rail services.
- **The evaluation period has been inspected repeatedly.** A new period is needed before claiming that improvements generalize.
- **Some alerts cannot be judged.** Of the reference system's 829 alerts, 190 were false alerts and 241 had insufficient observations to score. The 32.3% false-alert rate applies only to the 588 scoreable alerts.
- **An event is not the same as a commuter's experience.** Brief flowing intervals split congestion into separate events here. Whether those should count as one continuing disruption needs further study.
- **This was an offline evaluation.** Actual data delivery delays, route changes, notification fatigue, travel-time savings, and reduced lateness were not measured. Passing code tests does not establish those outcomes.

### 7. Where I'm Stopping, and What Comes Next

I'm keeping the random forest, score threshold of 0.5, and 30-minute suppression as the reference configuration and closing this phase of the project. I have enough evidence to explain why the recovery rules were not adopted. Continuing to change rules on the same data would not resolve the bigger question of whether this works in a new period or helps a commuter.

If I return to the project, I would prioritize:

1. **A new evaluation period.** Freeze the model and alert rule, check when observations actually become available, and evaluate without sending live notifications.
2. **The missing-input cases.** Find which required observations prevent predictions, then test whether a smaller-input model can recover useful opportunities.
3. **One bounded alert-rule experiment.** Allow early release after confirmed recovery, but release after 30 minutes even without it. This has not been tested, and earlier alerts could change later suppression, so improvement is not guaranteed.
4. **The original commuting question.** Identify when a warning would still allow a different route and what alert burden users would accept. Alternative-route and journey data would be needed to measure whether warnings actually help.

I would also like to explore whether earlier passenger-facing rail disruption information could support similar travel decisions. That would require a separate dataset, a definition of disruption, and a review of what information passengers already receive.

<div class="reflection" markdown="1">

### 8. Reflection

I started because I wanted information that might have helped me avoid a bad route choice. I expected most of the work to be about predicting traffic. In practice, a lot of it came down to deciding what counted as a useful warning, checking that time and missing data were handled correctly, and understanding why a prediction did not turn into an alert.

The most useful result was not always a higher detection rate. Finding that a reasonable-looking recovery rule blocked warnings gave me a clearer reason to keep the simpler approach. I have not built a system that can promise to prevent late arrivals, but I now have a more concrete understanding of what would need to work before making that claim.

</div>

*This page summarizes experiments completed through September 29, 2026. The latest recovery-rule experiment and loss diagnosis are recorded under `20260929T054130Z` in the project repository.*
