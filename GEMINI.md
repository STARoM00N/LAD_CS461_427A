# LogSentinel — CS461 Final Project (Gemini CLI Guide)

## Project Overview
**LogSentinel** is a Deep Learning-based Log Anomaly Detection system for Security Operations Centers (SOC).
- **Goal**: Binary classification at session level (`Normal = 0`, `Anomaly = 1`).
- **Primary Target**: F1-Score ≥ 80% on the HDFS dataset.
- **Prototype**: Interactive Streamlit web application for log analysis & anomaly explanation.

## Team
- ธนกร จำนงประโคน (1660701168)
- ศิวกร สุภาวงค์ (1660700251)
- นิธิสิทธิ์ เสนาหาญ (1660701317)
- ภูวดล ผิวบางกูล (1660702067)

## Tech Stack & Tools
- **Language**: Python 3.10+
- **Deep Learning / ML**: PyTorch, Scikit-learn
- **Log Parsing**: Drain3 (tree-based online parser with regex masking)
- **Data Manipulation**: Pandas, NumPy
- **Visualization**: Matplotlib, Seaborn
- **Web App**: Streamlit (Week 7)
- **Compute**: Local CUDA GPU / Google Colab
- **Tracking & Storage**: JSON metrics in `results/metrics/`, figures in `results/figures/`, weights in `checkpoints/`
- **VCS**: Git & GitHub (`origin/main`, feature branches)

## Repository Layout
```
app/            Streamlit prototype (Week 7 — not started)
checkpoints/    drain3_hdfs.bin, lstm_deeplog_hdfs.pt, deeplog_next_event_hdfs.pt, transformer_hdfs.pt
configs/        (empty) — YAML configs planned
data/raw/       HDFS.log, anomaly_label.csv (git-ignored)
data/processed/ HDFS/{train,val,test}.csv
docs/           Project form, progress presentation (HTML/MD/PDF), script
notebooks/      01_eda → 05_transformer (one notebook per week)
results/        metrics/*.json, figures/*.png
scripts/, src/  Package skeleton only — logic currently lives in notebooks
tests/          (empty)
```

## Datasets
1. **HDFS_v1** (Primary): 11,175,629 lines (~1.47 GiB), ~575,061 sessions grouped by `block_id`, ~2.9% anomaly.
   - Raw: `data/raw/HDFS/HDFS.log`, `data/raw/HDFS/preprocessed/anomaly_label.csv`
   - Splits (70/15/15 stratified): train 402,542 · val 86,259 · test 86,260
2. **BGL** (Secondary — Week 6): 4,747,963 lines from BlueGene/L (Loghub), for cross-system generalization.

## Roadmap & Status
| Week | Deliverable | Status |
|------|-------------|--------|
| 1 | EDA — `notebooks/01_eda.ipynb` | ✅ Done |
| 2 | Drain3 parsing (45 templates, 100% match with Loghub) — `02_log_parsing.ipynb` | ✅ Done |
| 3 | Baselines: Logistic Regression, Isolation Forest — `03_baseline_models.ipynb` | ✅ Done |
| 4 | Supervised LSTM — `04_lstm_deeplog.ipynb`; Pure DeepLog — `04b_deeplog_next_event.ipynb` | ✅ Done |
| 5 | Transformer Encoder + attention explainability — `05_transformer.ipynb`; DeepLog fixes in `04b` | ✅ Done |
| 6 | Model comparison + BGL cross-dataset — `06_comparison.ipynb` | ⏳ **Next** |
| 7 | Streamlit app — `app/app.py` | ⬜ Todo |
| 8 | Final report, presentation, demo | ⬜ Todo |

## Results So Far (HDFS test set, 86,260 sessions)
| Model | Paradigm | F1 | Precision | Recall | PR-AUC |
|-------|----------|----|-----------|--------|--------|
| Transformer Encoder (threshold 0.728) | Supervised | **99.04%** | 98.13% | 99.96% | 99.97% |
| Supervised Session LSTM | Supervised | 98.71% | 97.64% | 99.80% | 99.96% |
| Logistic Regression | Supervised | 98.02% | 96.12% | 100% | 97.53% |
| DeepLog + `<EOS>` + prob. threshold (A100, full data, τ=1.88e-4) | Normal-only | 92.78% | 95.94% | 89.83% | 98.42% |
| DeepLog original (top-k, k=5, w=10; A100 run, 44,703 sessions) | Normal-only | 67.93% | 90.34% | 54.43% | — |
| Isolation Forest | Unsupervised | 63.69% | 64.74% | 62.67% | 73.36% |

Notes:
- Transformer: 70,081 params, trained on CPU with full data (best epoch 4). 48 false alarms / 1 missed on test.
  Threshold rule (team decision, SOC recall priority): center of the widest contiguous region where validation F1 is
  within 0.01 of the best → 0.728 (plateau 0.465–0.990). The max-F1 point (0.99) was a one-point spike at the grid
  edge: F1 99.58% but 9 missed, with the weakest anomalies scoring just above the threshold.
- DeepLog fixes in `04b` (flags `USE_EOS`, `USE_PROB_THRESHOLD`; each combo saves to `deeplog_next_event[_eos][_prob]_hdfs.json`).
  More data alone did not help (68.69% → 67.93%). Two decision-rule problems did:
  (1) no end-of-session token, so early-terminated anomalous sessions looked normal;
  (2) top-k ranks beyond the 2–3 plausible events are near-random after longer training, so recall fell as k grew.
  Ablation on CPU (25k sessions × 5 epochs, test F1): baseline 68.69%, +prob 81.68%, +EOS 83.87%, both 91.34%.
  A100 run (EOS+prob, all 390,755 normal sessions): F1 92.78%, 96 false alarms / 257 missed. Checkpoint re-verified locally.
- Explainability (05 Part 3): last-layer [CLS] attention, heads averaged; `explain_session()` / `alert_explanation()` give the top events with Drain3 template text.
  Event IDs E1–E45 come from Drain3 (`checkpoints/drain3_hdfs.bin`), NOT Loghub's `HDFS.log_templates.csv` (29 templates, different numbering).
  Highest attention lift in anomalies: E34 (15.6×), E21 (14.6×), E33 (7.4×), E18, E16 — block errors/inconsistencies. E34/E33 also get high attention in normal sessions, so attention is a pointer for analysts, not proof.
  Figures 11–13: attention examples (TP/FN/FP/TN), lift by event type, per-head maps.
- Data integrity: an earlier Colab run read a partially uploaded train.csv (212,278 normals). Resolved; `04b` and `05` now
  stop with an error unless train/val/test have 402,542 / 86,259 / 86,260 rows. Local md5 of train.csv: f8aa54339c4ee65b2c4e8b1a4e3ffd94.
- Supervised LSTM threshold = 0.95 (chosen on validation F1), anomaly class weight ≈ 33.15.

## Evaluation Metrics (SOC Focused)
- **Primary**: F1-Score.
- **Secondary**: Recall (don't miss intrusions/failures), Precision (limit alert fatigue), PR-AUC, Confusion Matrix.
- **Accuracy Warning**: Never rely on Accuracy alone — predicting all Normal yields 97.1% accuracy (Accuracy Paradox).

## Coding Guidelines & Conventions
- Always include Python type hints (`from typing import ...`).
- Provide docstrings for all functions, classes, and pipeline steps.
- Organize notebooks with structured Markdown headers, Thai conceptual explanations, and visual charts.
- Reproducibility: fixed seed `SEED = 42` across Python, NumPy, and PyTorch.
- Save metrics as JSON in `results/metrics/<model>_hdfs.json` and figures in `results/figures/` (numbered prefix, e.g. `09_...png`).

## Teaching Mode (Strict Rule for AI Assistance)
- ทีมมาจากสาย Cyber Security ไม่มีพื้นฐาน ML/DL มาก่อน
- อธิบายหลักการ + เหตุผล (motivation) ก่อนเขียน code ทุกครั้ง
- ใช้ analogy ที่ทีมคุ้นเคย เช่น SOC, SIEM, Snort/YARA rules, IDS
- สอนทีละขั้น ไม่ dump code ก้อนใหญ่ที่ไม่มีคำอธิบาย
- ใช้ภาษาไทยอธิบายแนวคิด
