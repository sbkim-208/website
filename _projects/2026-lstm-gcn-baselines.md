---
title: "LSTM & GCN vs. Classical Baselines"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "From-scratch LSTM and GCN implementations in PyTorch, each benchmarked against a classical baseline (Random Forest, MLP) on the same task and split."
---

Two small PyTorch models, each built from scratch (no `torch_geometric`) and evaluated against a non-deep-learning baseline on the same task and split, rather than in isolation.

**Sequence forecasting — LSTM vs. Random Forest.** A single-layer LSTM (`hidden=32`) predicted next-hour bike-share demand from a 24-hour sliding window, with no hand-built lag/calendar features — the model has to learn the daily cycle purely from the raw sequence. A Random Forest on the same windows, flattened into 24 lag features, served as the classical baseline:

| Model | MAE | RMSE | R² |
|---|---|---|---|
| LSTM (raw sequence, no lag features) | 9.76 | 14.88 | 0.884 |
| Random Forest (24 flattened lag features) | 5.04 | 8.19 | **0.965** |

RF wins here — with a few months of hourly data, hand-built lag features plus a tree ensemble still beat a from-scratch LSTM. A follow-up hidden-size sweep (`[8, 16, 32, 64]`, test RMSE) checked whether the LSTM was simply under-capacity rather than fundamentally the wrong tool for this data size.

**Node prediction — GCN vs. MLP.** On a 16-node synthetic transit graph (main line + 2 branches + a cross-connection), the ridership target was generated to depend on a station's *neighbors'* transfer-count and parking values, not only its own — a case an MLP structurally can't solve. A 2-layer GCN (`H' = ReLU(Â·H·W)`) and a plain MLP were trained on the same 8 labeled nodes and evaluated on the other 8:

| Model | MAE | R² |
|---|---|---|
| MLP (no graph) | 105.94 | **−4.25** |
| GCN (uses `Â`) | 27.95 | **0.733** |

The MLP does worse than predicting the mean because it never sees neighbor information; the GCN propagates it through the normalized adjacency matrix and fits well. Together the two experiments make the same point from opposite sides: architecture choice only pays off when it matches what the data actually depends on — sequence order for the LSTM case (where RF's explicit lag features already captured that dependence better), and neighbor structure for the GCN case (where nothing but the graph could have captured it).
