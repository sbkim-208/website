---
title: "LSTM & GCN vs. Classical Baselines"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I built an LSTM and a GCN in PyTorch and compared them with simpler baselines on the same tasks and splits. Random Forest won the sequence task. The GCN did better on a small synthetic graph whose target depended on neighboring nodes."
---

It's easy to report a deep-learning R² by itself and call it a result. I wanted to see what happened when I put that score next to a Random Forest or a plain MLP on the same data. I built an LSTM and a GCN from scratch in PyTorch for two small comparisons.

## Predicting the next hour of bike-share demand
{: #study-scope }

The sequence task used a 24-hour sliding window to predict next-hour demand. A single-layer LSTM with `hidden=32` received the raw sequence, without hand-built lag or calendar features. Random Forest received those same values flattened into 24 lag inputs. Both used the same task and split.

The first comparison came out clearly in Random Forest's favor.

| Model | Test MAE | Test RMSE | Test R² |
| --- | ---: | ---: | ---: |
| LSTM, raw sequence | 9.76 | 14.88 | 0.884 |
| Random Forest, 24 lag inputs | 5.04 | 8.19 | 0.965 |

## Would a larger LSTM close the gap?
{: #study-process }

I didn't want to stop at that result. A smaller hidden layer might have been holding the LSTM back, so I tried 8, 16, 32, and 64 units in a separate set of runs.

| Hidden size | Test RMSE |
| ---: | ---: |
| 8 | 17.02 |
| 16 | 15.55 |
| 32 | 15.30 |
| 64 | 12.97 |

More capacity helped in this sweep, but even the 64-unit run stayed behind Random Forest's 8.19. The 32-unit result here is 15.30, not the initial run's 14.88. They came from different runs and shouldn't be treated as the same measurement.

This checked one possible explanation for the gap. It wasn't an exhaustive search over training choices, and it doesn't tell me that an LSTM would lose on another dataset or split.

## Giving the graph model information about neighbors
{: #study-results }

For the second experiment, I built a synthetic transit graph with 16 nodes, a main line, 2 branches, and a cross-connection. I generated a ridership target that depended on neighboring stations' transfer-count and parking values as well as the station itself.

A two-layer GCN and a plain MLP trained on the same 8 labeled nodes and were evaluated on the other 8. The GCN passed information through the normalized adjacency matrix, using `H' = ReLU(Â·H·W)`. The MLP received no graph information.

| Model | Evaluation MAE | Evaluation R² |
| --- | ---: | ---: |
| MLP, no graph | 105.94 | −4.25 |
| GCN, normalized adjacency | 27.95 | 0.733 |

Here the GCN did better. The MLP's negative R² means it did worse than predicting the evaluation target's mean. It had no direct access to the neighboring values used to construct the target, while the GCN could aggregate them.

That was a useful check of the implementation, but I had deliberately made a task where neighboring information mattered. With only 16 synthetic nodes, this doesn't establish how well either model would predict observed ridership.

## What I'd test before choosing a model
{: #study-next }

The sequence experiment gave me a reason to keep the simpler baseline for that setup. The graph experiment made me look more closely at what information each model could actually use. A good score was less interesting once I knew how the target had been constructed.

I'd repeat both comparisons across seeds and evaluation periods before drawing a broader conclusion. For the LSTM, other training settings are still worth checking against the same baseline. For the GCN, the next useful test would use observed transit data rather than a target I designed around the graph.
