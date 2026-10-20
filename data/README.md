# Data Directory

## Download Instructions

### HDFS_v1 (Primary Dataset)
1. Go to [Loghub HDFS](https://github.com/logpai/loghub/tree/master/HDFS)
2. Or download from [Zenodo](https://zenodo.org/records/8196385)
3. Place raw log files in `raw/HDFS/`
4. Required files:
   - `HDFS.log` — raw log file (11,175,629 lines, ~1.47 GiB)
   - `anomaly_label.csv` — session labels (Normal / Anomaly)

### BGL (Secondary Dataset)
1. Go to [Loghub BGL](https://github.com/logpai/loghub/tree/master/BGL)
2. Or download from [Zenodo](https://zenodo.org/records/8196385)
3. Place raw log files in `raw/BGL/`
4. Required files:
   - `BGL.log` — raw log file (4,747,963 lines, ~708 MiB)

### Quick Download
```bash
bash ../scripts/download_data.sh
```

## Directory Structure
```
data/
├── raw/            # Original downloaded files (do NOT modify)
│   ├── HDFS/
│   └── BGL/
├── processed/      # After Drain parsing & session creation
│   ├── HDFS/
│   └── BGL/
└── README.md       # This file
```

> ⚠️ Data files are excluded from git via `.gitignore` due to their large size.
