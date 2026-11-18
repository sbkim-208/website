---
title: "When, Not Where: Robotaxis in New York City"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I looked for the places where New York runs short of rides, and I found times of day instead. Using 240 million Uber and Lyft trips from 2025, I tested three assumptions, worked out how many trips a robotaxi needs to break even, and replayed real requests with extra vehicles added."
thumbnail: /assets/img/projects/nyc-robotaxi-saved-per-vehicle.png
hide_hero: true
---

I looked for the places where New York runs short of rides, and I found times of day instead. This report uses the 2025 trip records that New York City publishes for Uber and Lyft, about 240 million completed trips.

## Summary
{: #study-summary }

- **Stations.** Zones near subway stations had 3.4–3.5 times the trip density, but waits were about the same or shorter.
- **Busy and slow.** Only 4.4% of trips were both, less than the 6.25% expected by chance, and 81% of them fell in three time blocks.
- **Weather.** Rain added about 5 seconds to the wait and snow about 16. The gap between time blocks was 98 seconds.
- **Break-even.** A robotaxi that costs $100,000 a year needs about 14 trips a day; at $150,000 it needs 21.
- **AV versus driver.** I replayed real requests with AVs taking over trips in four long-wait zones. With 41% of its miles empty, one AV replaces $248 to $300 of driver pay in a weekday evening, against a fixed cost of $274 a day: about level. The answer turns on how much the AV drives empty ($205–$252 if it drives all the way back, $374–$479 if it never drives empty) and on what it costs.

## 1. Near stations: more rides, but not a longer wait
{: #study-scope }

I matched each pickup zone to its distance from the nearest subway station. In Manhattan almost every zone is within 500 m of a station, which means that stations are closer together than in other boroughs. So there are no far zones to compare with, and I made the comparison in the other boroughs. There, zones within 500 m of a station had 3.4–3.5 times the trip density of zones more than 1 km away, so the demand half of my assumption held. But average waits there were about the same or shorter, so the "not enough cars" half was not supported.

## 2. Busy and slow is a time of day, not a place
{: #study-process }

I decided that the top 25% of trips by density are “busy”, and the top 25% by long waits are “slow”. Even if busy and slow were unrelated, about 6 in every 100 trips would be both, simply because a quarter of a quarter is 6.25%. I found 4.4. One of the most interesting things is that the busier a place is, the shorter the wait tends to be. And when I looked at when those 4.4 happened, 81% of them fell in three time blocks: weekend nights after midnight, weekday nights, and weekday evenings.

![Share of busy-and-slow trips by time block: weekend midnight to 4 am 30%, weekday nights 30%, weekday 4 pm to 8 pm 22%, all other times 19%]({{ '/assets/img/projects/nyc-robotaxi-time-blocks.png' | relative_url }})

In these blocks 39% of trips waited more than 5 minutes 45 seconds, against 18% in other busy places, and the average wait was 98 seconds longer.

## 3. Weather adds seconds, not minutes
{: #study-results }

I also expected rain to make things worse. So I compared rainy days with other days in the same zone, day of the week, hour and month. On rainy days there were about 2% more rides, and people waited about 5 seconds longer. Snowy days added about 16 seconds and hot days about 8. One of the things I learned is that weather adds seconds, not minutes. That is much smaller than the 98-second gap I found between busy-and-slow trips and other busy trips. So my assumption that rain is when cars run short was not supported.

```
wait = b1·rain + b2·snow + b3·hot + b4·cold
       + fixed effect (zone × day of week × hour × month) + error
```

I built a regression to understand the relationship between wait time and weather. It compares only days in the same zone, day of the week, hour and month, so the difference that is left comes mostly from the weather. I decided to put four kinds of weather in it: rain, snow, hot days and cold days. Waits were about 5.4 seconds longer on rainy days, 16.3 on snowy days and 7.8 on hot days. Cold days showed no clear change (p = 0.74), so I cannot say that cold matters. To put that in context, the average wait is about 286 seconds, which means people who usually wait nearly 5 minutes have to wait about 5 seconds more when it rains. I also checked whether bad weather is worse in the three long-wait time blocks, and it was not clearly worse. But this does not mean weather does not matter. The 5 seconds is what is left after the platform has responded to rain, and riders who gave up are not in the data. I added hot and cold days after I saw the first results, so the hot-day result needs another check.

![Extra wait in seconds: long-wait time block +98, snowy day +16.3, hot day +7.8, rainy day +5.4, cold day +1.5]({{ '/assets/img/projects/nyc-robotaxi-weather.png' | relative_url }})

| Weather | Extra wait, seconds | Range from resampling days |
| --- | ---: | ---: |
| Rain (82 days) | +5.4 | +1.0 to +9.3 |
| Snow (23 days) | +16.3 | +6.0 to +25.4 |
| Hot (37 days) | +7.8 | +2.3 to +14.7 |
| Cold (23 days) | +1.5 | −8.0 to +11.0 |

## 4. How many trips a robotaxi needs
{: #study-breakeven }

If you operate a robotaxi business in New York, the first thing to check is one equation: trips needed per day = annual fixed cost ÷ 365 ÷ $19.56, where $19.56 is the driver pay you no longer pay on each trip, minus the cost of driving it. At $100,000 a year per vehicle that is 14 trips a day; at $150,000 it is 21. A human-driven vehicle averages 11.4 trips in about 6.4 hours on a working day, so an AV needs more trips per day than today’s cars. The place to start is the long-wait time blocks, where each hour with a passenger saves about 6.5% more, but that is only 4–7% of the vehicles on the road each day.

| Annual fixed cost per vehicle | Cost per day | Trips needed per day | If the vehicle is available 90% of days |
| --- | ---: | ---: | ---: |
| $100,000 | $274 | 14.0 | 15.6 |
| $150,000 | $411 | 21.0 | 23.3 |

Based on my hands-on experience as a train engineer at KORAIL, even the most durable and reliable vehicle cannot run every day of the year. In particular, trains often broke down even though engineers inspected them every morning. Therefore, in order to operate a robotaxi fleet reliably, companies should plan for inspection time and for vehicles that fail between inspections.

## 5. A virtual experiment: AVs instead of drivers
{: #study-experiment }

The first three sections did not find places where cars run short, so I could not build the case for AVs on filling a gap. The question that is left is a comparison: for the same trips, is it cheaper to pay a driver or to run an AV? To look at it, I designed an experiment: put 10, 20, 50 or 70 AVs into one zone in the long-wait time blocks and let them take over trips that drivers served. The trips are the same trips; no new riders appear. I measured two things. For the operator, the driver pay one AV replaces, compared with what the AV costs per day. For riders, the average wait and the share waiting more than 5 minutes 45 seconds.

### How it was run

I cannot put real vehicles on the street, so this was run as a replay. It takes the 4.76 million real trips of 2025 from four zones (Midtown Center, Times Sq/Theatre District, Lower East Side, Williamsburg North Side) and plays the requests back in the order they came in. An AV takes a rider only when it would arrive sooner than the real driver did. Human drivers are assumed not to react.

One rule changes the result more than anything else: what the AV does after it drops a rider off outside the zone. I ran three versions.

| Rule | What it means | Miles driven empty |
| --- | --- | ---: |
| Drives all the way back | The AV picks up only in its zone and returns empty after almost every trip. The lower edge | about 51% |
| Part of the way back (reference) | The AV needs about two thirds of the trip time before its next request. I set this so that 41% of its miles are empty, the share Waymo reports in California and the share the break-even used | 41% |
| Free again at drop-off | The AV is back in the zone the moment the rider gets out. This cannot happen; it is the upper edge | 3–7% |

### AV versus driver: replaced driver pay against fixed cost

For each AV I used the same idea as the break-even: what it leaves = driver pay on the trips it took over − its driving cost − its fixed cost per day.

![Driver pay one AV replaces in 8 evening hours under the three rules, against a fixed cost of $274 and $411 a day]({{ '/assets/img/projects/nyc-robotaxi-saved-per-vehicle.png' | relative_url }})

Weekday, 4 pm to midnight, 70 AVs in the zone, per AV:

| Zone | Trips in 8 hours (reference) | Drives all the way back | Reference, 41% empty | Free at drop-off |
| --- | ---: | ---: | ---: | ---: |
| Midtown Center | 12.0 | $245 | $294 | $466 |
| Times Sq / Theatre District | 11.6 | $252 | $300 | $479 |
| Lower East Side | 13.9 | $220 | $264 | $405 |
| Williamsburg North Side | 13.7 | $205 | $248 | $374 |

- **At 41% empty miles the AV and the driver cost about the same.** One AV replaces $248 to $300 of driver pay in the eight evening hours, against $274 a day ($100,000 a year). That is about $20 to $26 above the cost in Midtown Center and Times Sq, about $10 below in Lower East Side and $26 below in Williamsburg. None reaches $411 ($150,000 a year). Differences this small do not decide it.
- **The edges are far apart.** If the AV drives all the way back it replaces $205 to $252, and the driver is cheaper. If it never drove empty it would replace $374 to $479, and the AV would be cheaper.
- **So the answer turns on empty driving and on the assumed cost.** The riders, the fares and the zones are the same in all three versions. Only the time after a drop-off changes.
- **Only the evening is counted.** $274 is the cost of a whole day, and the pay replaced comes from eight evening hours on weekdays. What the AV does in the other 16 hours would come on top, and I did not look at it.
- **Evenings with longer waits.** On evenings with longer waits one AV replaced more pay, about $13 more for every extra minute of wait in the reference run (about 5%), although it took fewer trips. Trips are longer and pay the driver more on those evenings. This is a pattern across evenings, not an effect of waiting: it disappears once the evening's level of driver pay is held fixed.

### How many AVs

The number of AVs in a zone is a whole number n, and the best one makes this as large as possible: S(n) − c·n, where S(n) is the driver pay that n AVs replace in an evening and c is the fixed cost of one AV per day. S(n) has no formula, because it comes out of the replay, so I ran every n from 10 to 300 in steps of 10 and kept the best one. The rule this gives is simple: keep adding AVs until one more replaces less driver pay than it costs.

![Driver pay replaced by one more AV against AVs added, four zones, with fixed-cost lines at $274 and $411 a day]({{ '/assets/img/projects/nyc-robotaxi-fleet-size.png' | relative_url }})

In the reference run the curves sit within about $25 of the $274 line, so the number of AVs is not pinned down. In Midtown Center the best number is about 180 at $274 a day, about 250 at $249 and zero at $299. If the AV drives all the way back, the best number is zero in every zone. At the upper edge it would be about 90 to 260 per zone, and at that point the AVs would serve 70% to 80% of the zone's requests, which is where the assumption that drivers do not react is weakest.

### What riders get: shorter waits

The waits were measured a little differently from the money. For the money, AVs take over trips. For the waits, the real drivers are still there and AVs are added on top of them.

![Average wait against AVs added, weekday 4 pm to midnight, four zones, reference run]({{ '/assets/img/projects/nyc-robotaxi-wait-by-vehicles.png' | relative_url }})

Weekday, 4 pm to midnight, 70 AVs added, reference run:

| Zone | Average wait | Waiting over 5 min 45 s | Requests the AVs served |
| --- | ---: | ---: | ---: |
| Midtown Center | 320 → 286 s | 33.7% → 27.3% | 17% |
| Times Sq / Theatre District | 336 → 293 s | 36.2% → 28.4% | 20% |
| Lower East Side | 236 → 189 s | 16.8% → 10.2% | 35% |
| Williamsburg North Side | 223 → 186 s | 13.6% → 7.7% | 33% |

The average wait falls by about 5 to 7 seconds for every 10 AVs, and by 33 to 47 seconds with 70. That is less than half of the 98-second gap between time blocks. It takes many AVs to move the average, because one zone has 2,800 to 5,100 requests in an evening.

### What this does and does not show

With 41% of its miles empty and a cost of $100,000 a year, an AV that takes over evening trips in these zones costs about the same as the drivers it replaces. It is clearly cheaper only if it drives much less empty than that, or costs less. Shorter waits are a side effect, and they are measured in seconds.

It is not a forecast. In the replay a wait can only get shorter, so the direction is built in and only the sizes mean something. The AV is assumed to know how long the real driver would take, and drivers are assumed not to react. The real cost of an AV is not public. And I could not look at the other case, where AVs go where drivers are scarce and serve riders who would otherwise give up, because those riders are not in the data.

## 6. Limits
{: #study-limits }

This analysis only covers completed trips. Ride requests that were cancelled or abandoned are not in the data, so I measured rides, not demand. I also could not see how many drivers were working or where, so I cannot tell whether long waits come from more riders or fewer cars. The actual cost of an AV (purchase price, maintenance, charging time) is not public, so the fixed cost is an assumption, not an estimate. And because New York has no public AV operating data yet, I used California’s figure for the share of miles driven without a passenger. What’s more, station distance is measured from each zone’s centre point, not from where riders were actually picked up. Finally, the 25% cut-offs for "busy" and "slow" were my choice. With a stricter cut-off (the busiest 10%), busy places are no longer less slow.

## 7. How this was made
{: #study-next }

I wrote this report. I decided the questions, the order of the three assumptions, and the definitions. For example, I decided to cut “busy” and “slow” at the top 25%, I decided which kinds of weather go into the regression, and I set the design of the virtual experiment. I used an AI coding tool (Claude) to help write the SQL and Python and to draw the charts. I ran the aggregation scripts on my own computer and checked the results. In particular, I kept asking where each number came from. The first result said 3%, and when I asked, it came from two cut-offs that nobody had chosen for a reason, so I changed the definition.
