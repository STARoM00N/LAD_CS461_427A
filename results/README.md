# Results

## Structure
```
results/
├── figures/    # Generated plots (confusion matrix, PR curve, attention heatmap)
├── metrics/    # Experiment results in JSON/CSV format
└── README.md   # This file
```

## Metrics Format
Each experiment saves results as JSON:
```json
{
    "model": "lstm_deeplog",
    "dataset": "HDFS",
    "f1_score": 0.00,
    "precision": 0.00,
    "recall": 0.00,
    "pr_auc": 0.00,
    "hyperparameters": {},
    "timestamp": ""
}
```
