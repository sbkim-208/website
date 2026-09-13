---
title: "LSTM & GCN vs. Classical Baselines"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "From-scratch LSTM and GCN implementations in PyTorch, each held to the same bar: beat a classical baseline (Random Forest, MLP) on the identical task and split, or don't claim the win."
---

### 1. Overview

I built two small PyTorch models from scratch, an LSTM and a GCN, no `torch_geometric` since it needs CUDA I didn't have, and ran each one against a classical baseline (Random Forest, MLP) on the exact same task and split.

### 2. Problem

It's easy to report a deep-learning R² by itself and call it a result. It's a lot harder to trust once you put it next to what a Random Forest or a plain MLP gets on the same data. I wanted to know when a fancier model is actually earning its keep and when it's just a fancier way of doing worse.

### 3. Goals

Before running either experiment I made a rule for myself: never report a deep-learning result without a classical baseline on the exact same task and split. So the plan was to check whether an LSTM can beat a Random Forest at sequence forecasting when neither gets hand-built lag features, and whether a GCN beats an MLP on a node-prediction task built so the answer depends on a node's neighbors, not just its own attributes.

### 4. Process

For the sequence task, I trained a single-layer LSTM (`hidden=32`) to predict next-hour bike-share demand from a 24-hour sliding window, with no hand-built lag or calendar features, so it has to learn the daily cycle from the raw sequence alone. The Random Forest baseline got the same windows, just flattened into 24 lag features. When RF won clearly, I didn't want to just accept that, so I ran a follow-up hidden-size sweep on the LSTM to rule out the boring explanation, that it was under-capacity rather than the wrong tool for this amount of data.

For the node task, I built a 16-node synthetic transit graph, a main line plus 2 branches and a cross-connection, and generated the ridership target so it depends on a station's neighbors' transfer-count and parking values, not just its own. An MLP can't solve that no matter how it's trained. A 2-layer GCN (`H' = ReLU(Â·H·W)`) and a plain MLP were trained on the same 8 labeled nodes and evaluated on the other 8.

### 5. Result

First, RF beat the LSTM, clearly:

| Model                                      | MAE  | RMSE  | R²        |
| ------------------------------------------- | ---- | ----- | --------- |
| LSTM (raw sequence, no lag features)        | 9.76 | 14.88 | 0.884     |
| Random Forest (24 flattened lag features)   | 5.04 | 8.19  | **0.965** |

The capacity sweep I ran afterward:

| Hidden size | Test RMSE |
| ---: | ---: |
| 8  | 17.02 |
| 16 | 15.55 |
| 32 | 15.30 |
| 64 | 12.97 |

More capacity kept helping, but even at 64 units the LSTM's 12.97 doesn't get close to RF's 8.19. So it isn't a capacity problem. With this little data, hand-built lag features plus a tree ensemble just beat a from-scratch LSTM at every size I tried.

Second, on the node task, it went the other way:

| Model          | MAE    | R²        |
| -------------- | ------ | --------- |
| MLP (no graph) | 105.94 | **−4.25** |
| GCN (uses `Â`) | 27.95  | **0.733** |

The MLP did worse than just predicting the mean, since it can't see neighbor information at all. The GCN propagates it through the normalized adjacency matrix and fits well.

RF beat the LSTM because its lag features already captured the sequence dependence a small model couldn't learn on its own. The GCN beat the MLP because neighbor structure was the only thing that could explain the target in the first place. Knowing which case I was in before picking a model was really the point of both experiments.
