# Data Directory

> ⚠️ Data files are excluded from git via `.gitignore` due to their large size. Every team member must download them locally.

## HDFS_v1 (Primary Dataset)

Source: [Loghub](https://github.com/logpai/loghub) · [Zenodo record 8196385](https://zenodo.org/records/8196385)

### Download (Windows PowerShell — run from the project root)
```powershell
New-Item -ItemType Directory -Force ".\data\raw\HDFS"
curl.exe -L -o ".\data\raw\HDFS_v1.zip" "https://zenodo.org/records/8196385/files/HDFS_v1.zip?download=1"
tar -xf ".\data\raw\HDFS_v1.zip" -C ".\data\raw\HDFS"
Remove-Item ".\data\raw\HDFS_v1.zip"
```

### Download (Linux / macOS)
```bash
mkdir -p data/raw/HDFS
curl -L -o data/raw/HDFS_v1.zip "https://zenodo.org/records/8196385/files/HDFS_v1.zip?download=1"
unzip data/raw/HDFS_v1.zip -d data/raw/HDFS
rm data/raw/HDFS_v1.zip
```

### Required files
| File | Description |
|------|-------------|
| `raw/HDFS/HDFS.log` | Raw log file (11,175,629 lines, ~1.47 GiB) |
| `raw/HDFS/preprocessed/anomaly_label.csv` | Session labels (`BlockId`, `Label` = Normal / Anomaly) |
| `raw/HDFS/preprocessed/Event_traces.csv` | Loghub's reference event sequences — used to sanity-check our parsing |
| `raw/HDFS/preprocessed/HDFS.log_templates.csv` | Loghub's reference templates (29 events) |

> Use `HDFS.log`, **not** `HDFS_2k.log`. The 2k file is a random sample of lines, so sessions are incomplete (almost every session has only one event).

## BGL (Secondary Dataset — Week 6)
```powershell
New-Item -ItemType Directory -Force ".\data\raw\BGL"
curl.exe -L -o ".\data\raw\BGL.zip" "https://zenodo.org/records/8196385/files/BGL.zip?download=1"
tar -xf ".\data\raw\BGL.zip" -C ".\data\raw\BGL"
Remove-Item ".\data\raw\BGL.zip"
```
Required file: `raw/BGL/BGL.log` (4,747,963 lines, ~708 MiB)

## Directory Structure
```
data/
├── raw/                    # Original downloaded files (do NOT modify)
│   ├── HDFS/
│   │   ├── HDFS.log
│   │   └── preprocessed/
│   │       ├── anomaly_label.csv
│   │       ├── Event_traces.csv
│   │       └── HDFS.log_templates.csv
│   └── BGL/
├── processed/              # Created by notebooks/02_log_parsing.ipynb
│   └── HDFS/
│       ├── train.csv       # 70%
│       ├── val.csv         # 15%
│       └── test.csv        # 15%
└── README.md               # This file
```

Processed CSV columns: `BlockId`, `Sequence` (event IDs separated by spaces, e.g. `E5 E22 E5 E11`), `Label` (0 = Normal, 1 = Anomaly).
