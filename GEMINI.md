# LogSentinel — CS461 Final Project

## Project Overview
**LogSentinel** is a Deep Learning-based Log Anomaly Detection system for Security Operations Centers (SOC).
- Binary classification at session level: Normal / Anomaly
- Target F1-Score ≥ 80% on HDFS dataset
- Prototype web app via Streamlit

## Team
- ธนกร จำนงประโคน (1660701168)
- ศิวกร สุภาวงค์ (1660700251)
- นิธิสิทธิ์ เสนาหาญ (1660701317)
- ภูวดล ผิวบางกูล (1660702067)

## Tech Stack
- **Language**: Python
- **ML/DL**: PyTorch, Scikit-learn
- **Log Parsing**: Drain3
- **Data**: Pandas, NumPy
- **Visualization**: Matplotlib, Seaborn
- **Prototype**: Streamlit
- **Compute**: Google Colab / GPU
- **VCS**: Git, GitHub

## Datasets
- **HDFS_v1** (primary): 11.17M lines, ~575K sessions, ~2.9% anomaly from Loghub
- **BGL** (secondary): 4.75M lines from BlueGene/L supercomputer from Loghub

## Models
1. **Baseline**: Isolation Forest, Logistic Regression (event count vectors)
2. **LSTM** (DeepLog approach): next-event prediction, top-k anomaly detection
3. **Transformer Encoder**: self-attention on log sequences, attention-based explainability

## Evaluation Metrics
- Primary: F1-Score
- Secondary: Precision, Recall, PR-AUC, Confusion Matrix

## Coding Guidelines
- Use Python type hints
- Write docstrings for all public functions and classes
- Keep notebooks organized with clear markdown headers
- Use reproducible random seeds for experiments
- Log hyperparameters and results systematically
