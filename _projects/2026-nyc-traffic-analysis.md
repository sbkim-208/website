---
title: "FDR Drive Northbound: Could I Warn a Driver Before Exit 2?"
category: other
year: 2026
summary: "I tested congestion warnings on northbound FDR Drive, then built a replay to see whether a warning would arrive before a driver passed Exit 2."
thumbnail: /assets/img/projects/fdr-217-exit-replay.svg
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/nyc_traffic_data_analysis
---

During my commute to work, I sometimes found out about traffic only after passing the turn where I could have taken another route. By then, knowing about the congestion didn't help much. Could road speeds tell me a slowdown was coming while I still had a choice?

I narrowed that question to northbound FDR segment 217, from Catherine Slip to 25th Street. Exit 2 is the turn a driver would have to take before reaching that stretch.

## The missing readings mattered
{: #fdr-data }

The starting point was 4,233,169 NYC DOT sensor records from April through July 2024. Before trying to predict anything, I checked how often a recorded reading had no `-101` error flag.

| Borough | Records without the error flag |
| --- | ---: |
| Queens | 93.4% |
| Bronx | 83.7% |
| Staten Island | 65.1% |
| Brooklyn | 62.1% |
| Manhattan | 56.3% |

Manhattan, where the FDR runs, had the lowest share. These percentages describe the records I received. They don't account for readings that never arrived.

Those gaps also exposed a problem in my evaluation. At first, I used the sixth row ahead as the speed 30 minutes later. That only works if every reading is five minutes apart. I changed the code to match timestamps, so a 9:00 prediction was checked against 9:30. If the future observation was missing, I left it missing instead of filling in the answer.

The [original sensor map]({{ '/assets/interactive/congestion_error_map.html' | relative_url }}) shows the errors and evening congestion across NYC. I've also kept the [earlier analysis and its charts]({{ '/projects/2026-nyc-traffic-analysis/original-study/' | relative_url }}), which covers both FDR study segments.

## Predicting a slowdown before it starts
{: #fdr-method }

My first model predicted speed exactly 30 minutes ahead. But traffic could slow after 10 minutes and recover by minute 30. The final speed would look fine even though a driver had sat through the slowdown.

I changed the target to whether congestion would *begin* at any point in the next half hour, while the road was still flowing. For these onset experiments, I trained on 2023 data, compared models using 2024, and checked the warnings against 2025 events. I had already inspected 2025 in earlier work, so this was an exploratory evaluation.

I also tried waiting for five minutes of recovery before allowing another alert. On segment 217, that reduced the number of events caught at least 10 minutes early from 99 to 86 out of the same 501 events. I kept the original rule, which suppresses repeat alerts for 30 minutes after a warning.

## Replaying the first quarter of 2026
{: #fdr-2026 }

For the weekday replay from January through March 2026, I used a random forest that gave more training weight to slowdowns beginning within 5 or 10 minutes. This differs from the reference model used in the 2025 rule comparison above. I held the settings fixed during the replay, though I had already looked at this quarter in earlier work.

Of 290 congestion events on segment 217, the model warned about 101 between 5 and 30 minutes before they began. Only 53 of those events received at least 10 minutes' notice.

Looking at the alerts themselves gives a different denominator:

| Outcome | Alerts |
| --- | ---: |
| Matched to a later slowdown | 101 |
| False alarm | 15 |
| Could not be scored because observations were missing | 21 |
| Total issued | 137 |

There were also times when the model couldn't run. Of 18,432 scheduled five-minute checks, 1,738 showed that the road was flowing. Only 704 of those had enough recent readings for a prediction. The other 1,034 did not. The simulator shows those cases separately because silence could otherwise look like an all-clear.

To check the replay, I ran the saved model again on all 704 inputs in the original feature order. Every score matched within 10⁻¹². The [model version and monthly counts]({{ '/assets/interactive/fdr-model-check.json' | relative_url }}) are available alongside the demo.

## Would the driver still reach the exit in time?
{: #fdr-exit-demo }

On January 26, 2026, at 11:50 p.m., segment 217 was flowing and the replay issued a warning. Five minutes later, the recorded speed fell below the congestion threshold. That gives the model five minutes of lead time. It doesn't tell me where a driver was in relation to Exit 2.

The [Exit 2 simulator]({{ '/assets/interactive/fdr-exit-2-replay.html' | relative_url }}) lets you try that part separately. Give the driver 60 seconds to reach the exit and set the notification delay to 10 seconds. The warning arrives with 50 seconds left. Change the delay to 90 seconds and the driver has already passed it.

The road predictions and later speeds come from the recorded study. The route, time to the exit, and notification delay are settings you choose. I haven't measured actual phone delivery times or the time a detour would save. The replay gets me closer to the commuting question I started with, but those pieces would still need to be measured.
