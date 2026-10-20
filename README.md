# LogSentinel

> **Deep Learning-based Log Anomaly Detection for Security Operations Centers**

ระบบตรวจจับความผิดปกติใน System Log ด้วย Deep Learning สำหรับศูนย์ปฏิบัติการความปลอดภัยไซเบอร์ (SOC)

## Overview

LogSentinel ใช้ Deep Learning ตรวจจับ session ที่ผิดปกติจาก system log โดยอัตโนมัติ:
- **Binary Classification**: Normal / Anomaly ระดับ session
- **Target**: F1-Score ≥ 80% บนชุดข้อมูล HDFS
- **Prototype**: Streamlit web app สำหรับอัปโหลด log แล้วดูผลการทำนาย

## Models

| Model | Approach | Description |
|-------|----------|-------------|
| Isolation Forest | Baseline | Unsupervised anomaly detection on event count vectors |
| Logistic Regression | Baseline | Supervised classification on event count vectors |
| LSTM (DeepLog) | Deep Learning | Next-event prediction, top-k anomaly detection |
| Transformer Encoder | Deep Learning | Self-attention on log sequences, attention-based explainability |

## Project Structure

```
LogSentinel/
├── data/               # Datasets (HDFS, BGL) — not committed
├── notebooks/          # Jupyter Notebooks (EDA → Training → Comparison)
├── src/                # Reusable Python source code
│   ├── data/           #   Data loading & preprocessing
│   ├── models/         #   Model definitions
│   ├── training/       #   Training & evaluation logic
│   └── utils/          #   Shared utilities
├── configs/            # Experiment configurations (YAML)
├── checkpoints/        # Saved model weights — not committed
├── results/            # Experiment results & figures
├── app/                # Streamlit web application
├── scripts/            # Utility scripts
├── tests/              # Unit tests
└── docs/               # Documentation & project forms
```

## Setup

```bash
# Clone the repository
git clone https://github.com/<your-org>/LogSentinel.git
cd LogSentinel

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac

# Install dependencies
pip install -r requirements.txt

# Download datasets
bash scripts/download_data.sh
```

## Usage

### Training (via notebooks)
Open notebooks in order (`01_eda.ipynb` → `06_comparison.ipynb`) in Jupyter or Google Colab.

### Streamlit App
```bash
streamlit run app/app.py
```

## Datasets

| Dataset | Lines | Sessions | Anomaly Rate | Source |
|---------|-------|----------|-------------|--------|
| HDFS_v1 | 11,175,629 | ~575,061 | ~2.9% | [Loghub](https://github.com/logpai/loghub) |
| BGL | 4,747,963 | — | — | [Loghub](https://github.com/logpai/loghub) |

## Team

- ธนกร จำนงประโคน (1660701168)
- ศิวกร สุภาวงค์ (1660700251)
- นิธิสิทธิ์ เสนาหาญ (1660701317)
- ภูวดล ผิวบางกูล (1660702067)

## References

1. Du, M. et al. (2017). *DeepLog: Anomaly Detection and Diagnosis from System Logs through Deep Learning.* ACM CCS 2017.
2. Guo, H. et al. (2021). *LogBERT: Log Anomaly Detection via BERT.* IJCNN 2021.
3. He, P. et al. (2017). *Drain: An Online Log Parsing Approach with Fixed Depth Tree.* IEEE ICWS 2017.
4. Zhu, J. et al. (2023). *Loghub: A Large Collection of System Log Datasets.* IEEE ISSRE 2023.

## License

CS461 — Deep Learning | Final Project
