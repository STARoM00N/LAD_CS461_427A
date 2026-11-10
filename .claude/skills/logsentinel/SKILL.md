---
name: logsentinel
description: Specialized workflows for LogSentinel (Deep Learning-based Log Anomaly Detection for SOC) — covering Drain3 parsing, baseline ML, LSTM DeepLog next-event prediction, Transformer encoder, and SOC metrics evaluation.
---

# LogSentinel Agent Skill

## Purpose
This skill equips Claude Code with domain knowledge and execution procedures for the **LogSentinel** project (CS461 Final Project: Deep Learning Log Anomaly Detection for Security Operations Centers).

## Key Workflow Runbooks

### 1. Data Pipeline & File Organization
```
Final_Project/
├── data/
│   ├── raw/HDFS/                     # Raw HDFS.log (11.17M lines) & labels
│   ├── processed/HDFS/               # train.csv (70%), val.csv (15%), test.csv (15%)
├── notebooks/
│   ├── 01_eda.ipynb                  # Exploratory Data Analysis
│   ├── 02_log_parsing.ipynb          # Drain3 template mining (45 events)
│   ├── 03_baseline_models.ipynb      # Logistic Regression & Isolation Forest
│   ├── 04_lstm_deeplog.ipynb         # Supervised Session-level LSTM
│   ├── 04b_deeplog_next_event.ipynb  # Unsupervised DeepLog Next-Event Prediction
│   ├── 05_transformer.ipynb          # Transformer Encoder with Self-Attention
│   ├── 06_comparison.ipynb           # Model comparison & BGL cross-dataset test
├── results/
│   ├── figures/                      # Confusion matrices & loss curves
│   └── metrics/                      # JSON metric records per model
└── checkpoints/                      # Saved PyTorch checkpoints & Drain3 state
```

### 2. DeepLog Paradigm vs Supervised LSTM
When working on Week 4 models, distinguish clearly between the two paradigms:
- **04 Supervised Session-level LSTM**:
  - Input: Entire variable-length session sequence $[e_1, e_2, \dots, e_L]$
  - Target: Binary label ($0 = \text{Normal}, 1 = \text{Anomaly}$)
  - Loss: `nn.BCEWithLogitsLoss(pos_weight=33.15)`
  - Threshold: Selected on validation F1-score (~0.95)
- **04b Pure DeepLog (Next-Event Prediction)**:
  - Concept: Model how normal execution flows from step to step (similar to predictive text).
  - Training: **Normal sessions only** (`train[train['Label'] == 0]`). Zero attack labels needed.
  - Window size $h$: Preceding $h$ events (e.g., $h=10$) predict the probability distribution over all event templates.
  - Anomaly Rule: If actual event $e_t$ is not in the Top-$k$ candidates (e.g. $k=9$), mark event as anomalous. If anomalous events $\ge 1$ in session, flag session as Anomaly.
  - Loss: `nn.CrossEntropyLoss()`

### 3. Evaluation & Metrics Guidelines
- **Always prioritize F1-Score**: In HDFS, Anomaly is ~2.9%. Accuracy is deceptive (Accuracy Paradox).
- **Precision**: Measures alert reliability ($\text{True Positive} / (\text{True Positive} + \text{False Positive})$). Prevents SOC alert fatigue.
- **Recall**: Measures detection completeness ($\text{True Positive} / (\text{True Positive} + \text{False Negative})$). Critical: missing attacks is dangerous.
- **PR-AUC**: Measures precision-recall trade-off across all probability thresholds.
- **Results Format**: Save all metrics systematically to `results/metrics/<model>_hdfs.json` matching the schema:
  ```json
  {
      "model": "model_name",
      "dataset": "HDFS",
      "f1_score": 0.0000,
      "precision": 0.0000,
      "recall": 0.0000,
      "pr_auc": 0.0000,
      "hyperparameters": {},
      "timestamp": "ISO-8601"
  }
  ```

### 4. Git & Branching Rules
- Never commit broken code to `main`.
- Use feature branches: `git checkout -b <feat|fix>/<name>`
- Verify `data/processed/` and `checkpoints/` large files are excluded via `.gitignore`.
