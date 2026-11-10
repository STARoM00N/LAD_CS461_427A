# LogSentinel — CS461 Final Project (Claude Code Guide)

## Project Overview
**LogSentinel** is a Deep Learning-based Log Anomaly Detection system for Security Operations Centers (SOC).
- **Goal**: Binary classification at session level (`Normal = 0`, `Anomaly = 1`).
- **Primary Target**: F1-Score $\ge 80\%$ on the HDFS dataset.
- **Prototype**: Interactive Streamlit web application for log analysis & anomaly explanation.

## Team
- ธนกร จำนงประโคน (1660701168)
- ศิวกร สุภาวงค์ (1660700251)
- นิธิสิทธิ์ เสนาหาญ (1660701317)
- ภูวดล ผิวบางกูล (1660702067)

## Tech Stack & Tools
- **Language**: Python 3.10+
- **Deep Learning / ML**: PyTorch, Scikit-learn
- **Log Parsing**: Drain3 (Tree-based online parser with regex masking)
- **Data Manipulation**: Pandas, NumPy
- **Visualization**: Matplotlib, Seaborn
- **Web App**: Streamlit (Week 7)
- **Tracking & Storage**: JSON metric records in `results/metrics/`, figures in `results/figures/`, model weights in `checkpoints/`
- **VCS**: Git & GitHub (`origin/main`, feature branches)

## Datasets
1. **HDFS_v1** (Primary):
   - 11,175,629 lines (~1.47 GiB), ~575,061 sessions grouped by `block_id`.
   - Highly imbalanced: ~2.9% Anomaly sessions.
   - Raw location: `data/raw/HDFS/HDFS.log` & `data/raw/HDFS/preprocessed/anomaly_label.csv`
   - Processed splits: `data/processed/HDFS/{train,val,test}.csv` (70% / 15% / 15% stratified split).
2. **BGL** (Secondary — Week 6):
   - 4,747,963 lines from BlueGene/L supercomputer from Loghub.
   - Used for cross-system generalization testing.

## Model Progression & Roadmap
1. **Week 1 — EDA** (`notebooks/01_eda.ipynb`):
   - Explored log format, component distributions, session lengths, and imbalanced labels (~2.9%).
2. **Week 2 — Log Parsing** (`notebooks/02_log_parsing.ipynb`):
   - Drain3 parsing with regex masking (`<BLK>`, `<IP>`, `<NUM>`).
   - Mined 45 event templates ($E_1 \dots E_{45}$), validated 100% against Loghub reference traces.
   - Stratified train (402,542), validation (86,259), test (86,260) splits saved to `data/processed/HDFS/`.
3. **Week 3 — Baseline Models** (`notebooks/03_baseline_models.ipynb`):
   - Event Count Vector representations via `CountVectorizer`.
   - **Logistic Regression** (Supervised): F1 = 98.02%, Recall = 100%, Precision = 96.12%, PR-AUC = 97.53%.
   - **Isolation Forest** (Unsupervised): F1 = 63.69%, Recall = 62.67%, Precision = 64.74%, PR-AUC = 73.36%.
4. **Week 4 — LSTM Models**:
   - `04_lstm_deeplog.ipynb`: Supervised Session-level LSTM Classifier with class-weighted BCEWithLogitsLoss.
   - `04b_deeplog_next_event.ipynb`: Pure DeepLog (Du et al., 2017) — trained on Normal sessions only with sliding-window Next-Event Prediction and Top-$k$ candidate detection.
5. **Week 5 — Transformer Encoder** (`notebooks/05_transformer.ipynb`):
   - Multi-head self-attention on log sequences with attention-based explainability heatmaps for SOC analysts.
6. **Week 6 — Model Comparison & Cross-Dataset Evaluation** (`notebooks/06_comparison.ipynb`):
   - Compare all models on HDFS and evaluate transferability on the BGL dataset.
7. **Week 7 — Web Application Prototype** (`app/app.py`):
   - Streamlit UI for log upload, session inference, and highlight of suspicious events.
8. **Week 8 — Final Report, Presentation, & Demo**.

## Evaluation Metrics (SOC Focused)
- **Primary**: F1-Score (Harmonic mean of Precision and Recall).
- **Secondary**: Recall (critical for SOC: must not miss true intrusions/failures), Precision (minimizes alert fatigue), PR-AUC (evaluates model across all thresholds), and Confusion Matrix.
- **Accuracy Warning**: Never rely on Accuracy alone due to the *Accuracy Paradox* (a model predicting 100% Normal achieves 97.1% accuracy while letting all attacks breach the network).

## Coding Guidelines & Conventions
- Always include Python type hints (`from typing import ...`).
- Provide docstrings for all functions, classes, and pipeline steps.
- Organize notebooks with structured Markdown headers, Thai conceptual explanations, and visual charts.
- Maintain reproducibility: use fixed random seed (`SEED = 42`) across Python, NumPy, and PyTorch.
- Always save metric summaries as JSON in `results/metrics/<model>_hdfs.json` and figures in `results/figures/`.

## Teaching Mode (Strict Rule for AI Assistance)
- The team has a strong Cyber Security background but is new to Machine Learning & Deep Learning.
- Always explain the theoretical concept, motivation, and analogies (e.g., comparing ML concepts to SOC, SIEM, Snort/YARA, IDS) before writing code.
- Explain step-by-step; avoid dumping unannotated blocks of code.
- Use clear Thai explanations for conceptual clarity.
