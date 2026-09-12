---
title: "LSTM & GCN vs. Classical Baselines"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "From-scratch LSTM and GCN implementations in PyTorch, each held to the same bar: beat a classical baseline (Random Forest, MLP) on the identical task and split, or don't claim the win."
---

I built two small PyTorch models from scratch — no `torch_geometric`, since it needs CUDA I didn't have — and made a rule for myself before running either one: never report a deep-learning result without a classical baseline on the exact same task and split. Trusting an R² in isolation is easy; trusting it next to what a Random Forest or a plain MLP gets on the same data is a much higher bar, and it's the bar these two experiments are held to.

**Sequence forecasting — LSTM vs. Random Forest.** A single-layer LSTM (`hidden=32`) predicted next-hour bike-share demand from a 24-hour sliding window, with no hand-built lag/calendar features — the model has to learn the daily cycle purely from the raw sequence. A Random Forest on the same windows, flattened into 24 lag features, served as the baseline:

| Model                                     | MAE  | RMSE  | R²        |
| ----------------------------------------- | ---- | ----- | --------- |
| LSTM (raw sequence, no lag features)      | 9.76 | 14.88 | 0.884     |
| Random Forest (24 flattened lag features) | 5.04 | 8.19  | **0.965** |

RF wins, clearly — and I didn't take that at face value either. I ran a follow-up hidden-size sweep specifically to rule out the boring explanation, that the LSTM was simply under-capacity rather than the wrong tool for a few months of hourly data:

| Hidden size | Test RMSE |
|---:|---:|
| 8 | 17.02 |
| 16 | 15.55 |
| 32 | 15.30 |
| 64 | 12.97 |

More capacity keeps helping — but even at 64 units the LSTM's 12.97 doesn't touch RF's 8.19. It isn't a capacity problem: with this little data, hand-built lag features plus a tree ensemble beat a from-scratch LSTM at every size I tested.

**Node prediction — GCN vs. MLP.** On a 16-node synthetic transit graph (main line + 2 branches + a cross-connection), the ridership target was generated to depend on a station's _neighbors'_ transfer-count and parking values, not only its own — a case an MLP structurally can't solve. A 2-layer GCN (`H' = ReLU(Â·H·W)`) and a plain MLP were trained on the same 8 labeled nodes and evaluated on the other 8:

| Model          | MAE    | R²        |
| -------------- | ------ | --------- |
| MLP (no graph) | 105.94 | **−4.25** |
| GCN (uses `Â`) | 27.95  | **0.733** |

The MLP does worse than predicting the mean, because it structurally cannot see neighbor information; the GCN propagates it through the normalized adjacency matrix and fits well. Put the two results side by side and they make the same point from opposite directions: a fancier architecture only pays for itself when it matches what the data actually depends on. RF beat the LSTM because its lag features already captured the sequence dependence a small model couldn't learn from raw data alone. The GCN beat the MLP because neighbor structure was the *only* thing that could explain the target — no amount of feature engineering on a single node's own attributes would have recovered it. Knowing which case you're in before you pick a model is the actual skill; the benchmark numbers are just how I checked I'd picked correctly.
