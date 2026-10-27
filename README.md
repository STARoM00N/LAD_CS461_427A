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
LAD_CS461_427A/
├── data/               # Datasets (HDFS, BGL) — not committed, see data/README.md
├── notebooks/          # Jupyter Notebooks (EDA → Training → Comparison)
├── src/                # Reusable Python source code (planned)
│   ├── data/           #   Data loading & preprocessing
│   ├── models/         #   Model definitions
│   ├── training/       #   Training & evaluation logic
│   └── utils/          #   Shared utilities
├── checkpoints/        # Drain3 state & model weights — not committed
├── results/            # Experiment results
│   ├── figures/        #   Plots (EDA, confusion matrices, ...)
│   └── metrics/        #   Metrics per model (JSON)
├── app/                # Streamlit web application (planned — Week 7)
├── tests/              # Unit tests
└── docs/               # Documentation & project forms
```

## Setup

```bash
# Clone the repository
git clone https://github.com/STARoM00N/LAD_CS461_427A.git
cd LAD_CS461_427A

# Create virtual environment
python -m venv venv
source venv/bin/activate      # Linux/Mac
# venv\Scripts\Activate.ps1   # Windows PowerShell

# Install dependencies
pip install -r requirements.txt
```

### Download datasets
Datasets are not in git (too large). Follow the steps in [`data/README.md`](data/README.md) to download **HDFS_v1** from Zenodo and place it in `data/raw/HDFS/`.

> ⚠️ Use the full `HDFS.log` (11.17M lines), **not** `HDFS_2k.log`. The 2k file is a random sample of lines, so sessions are incomplete.

## Usage

### Training (via notebooks)
Open notebooks in order in Jupyter / VS Code. Each notebook reads the output of the previous one.

| Notebook | Status |
|----------|--------|
| `01_eda.ipynb` | ✅ Done |
| `02_log_parsing.ipynb` | ✅ Done — outputs `data/processed/HDFS/{train,val,test}.csv` and `checkpoints/drain3_hdfs.bin` |
| `03_baseline_models.ipynb` | ✅ Done — outputs `results/metrics/*.json` |
| `04_lstm_deeplog.ipynb` | ⏳ Week 4 |
| `05_transformer.ipynb` | ⏳ Week 5 |
| `06_comparison.ipynb` | ⏳ Week 6 |

### Streamlit App (planned — Week 7)
```bash
streamlit run app/app.py
```

## Git Workflow

Don't commit directly to `main`. Create a branch for each piece of work, then open a Pull Request:

```bash
git checkout main && git pull origin main
git checkout -b <type>/<short-name>      # e.g. feat/lstm-deeplog, fix/log-parsing
# ... work, then:
git add <files>
git commit -m "Short description"
git push -u origin <type>/<short-name>
# open a Pull Request on GitHub → review → merge
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
