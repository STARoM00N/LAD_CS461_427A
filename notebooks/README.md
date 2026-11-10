# Notebooks

Jupyter Notebooks ของโปรเจกต์ LogSentinel เรียงตามลำดับการทำงาน (ทุก notebook ใช้ path แบบ relative เช่น `../data/...` จึงต้องรันจากโฟลเดอร์ `notebooks/`)

## สถานะความคืบหน้า

| # | Notebook | สถานะ | เนื้อหา |
|---|----------|-------|---------|
| 1 | `01_eda.ipynb` | ✅ รันแล้ว | สำรวจ HDFS log: รูปแบบบรรทัด, log level, label, ความยาว session |
| 2 | `02_log_parsing.ipynb` | ✅ รันแล้ว | Drain3 parsing + masking, สร้าง session ตาม block_id, แบ่ง train/val/test |
| 3 | `03_baseline_models.ipynb` | ✅ รันแล้ว | Event Count Vector + Logistic Regression / Isolation Forest |
| 4 | `04_lstm_deeplog.ipynb` | ✅ รันแล้ว | Supervised session-level LSTM classifier |
| 4b | `04b_deeplog_next_event.ipynb` | ✅ รันแล้ว | DeepLog (Du et al., 2017): Next-Event Prediction + Top-k, ฝึกด้วย Normal session เท่านั้น |
| 5 | `05_transformer.ipynb` | ⏳ ยังไม่ได้สร้าง | Transformer Encoder (ตาม roadmap) |
| 6 | `06_comparison.ipynb` | ⏳ ยังไม่ได้สร้าง | เปรียบเทียบโมเดล + ทดสอบบน BGL (ตาม roadmap) |

## สรุปผล HDFS Test Set (86,260 sessions, Anomaly 2,526)

ค่าจาก `results/metrics/*.json` และ output ของ notebook

| โมเดล | Notebook | Precision | Recall | F1 | PR-AUC | TN / FP / FN / TP |
|-------|----------|-----------|--------|----|--------|-------------------|
| Logistic Regression | 03 | 96.12% | 100.00% | 98.02% | 97.53% | 83,632 / 102 / 0 / 2,526 |
| Isolation Forest | 03 | 64.74% | 62.67% | 63.69% | 73.36% | 82,872 / 862 / 943 / 1,583 |
| Supervised LSTM | 04 | 97.64% | 99.80% | 98.71% | 99.96% | 83,673 / 61 / 5 / 2,521 |
| DeepLog (k = 5) | 04b | 86.77% | 56.85% | 68.69% | — | 83,515 / 219 / 1,090 / 1,436 |

---

## 01 — EDA (`01_eda.ipynb`)

**Input:** `data/raw/HDFS/HDFS.log`, `data/raw/HDFS/preprocessed/anomaly_label.csv`, `HDFS.log_templates.csv` และ `Event_traces.csv` ของ Loghub (ใน `preprocessed/`)

**สิ่งที่ทำและผลที่ได้**
- อ่าน log ทั้งไฟล์: **11,175,629 บรรทัด**, parse ไม่ได้ 0 บรรทัด, block_id ไม่ซ้ำ **575,061** (= จำนวน session)
- Log level: `INFO` 10,812,836 (96.75%), `WARN` 362,793 (3.25%)
- Label: Normal 558,223 (97.07%), Anomaly 16,838 (2.93%)
- ความยาว session (นับจากบรรทัด log): min 2, max 310, mean 21.9, median 22, P95 = 31, P99 = 36
- ค่าเฉลี่ยความยาว: Normal 22.0, Anomaly 19.2
- ไฟล์ templates ของ Loghub มี 29 event types

**Output (figures):** `01_log_level_distribution.png`, `02_label_distribution.png`, `03_session_length_distribution.png`, `04_session_length_normal_vs_anomaly.png`

---

## 02 — Log Parsing & Session Creation (`02_log_parsing.ipynb`)

**Input:** `HDFS.log` (ไฟล์เต็ม), `anomaly_label.csv`, `Event_traces.csv` (ใช้ตรวจผล)

**สิ่งที่ทำ**
- ใช้ Drain3 `TemplateMiner` + `FilePersistence` (บันทึก state ไว้ให้ Streamlit app ใช้ต่อ)
- Masking ก่อน parse:
  - `blk_-?\d+` → `<BLK>`
  - `(\d{1,3}\.){3}\d{1,3}(:\d+)?` → `<IP>`
  - `\b\d+\b` → `<NUM>`
- ดึง content ด้วย regex แล้วรวม event ID ตาม block_id เป็น sequence
- Merge กับ label (Normal = 0, Anomaly = 1)
- แบ่งข้อมูลด้วย `train_test_split` แบบ stratify ตาม Label, `random_state=42`: 70% train แล้วแบ่ง 30% ที่เหลือเป็น val / test อย่างละครึ่ง

**ผลที่ได้**
- Parse 11,175,629 บรรทัด (parse ไม่ได้ 0), ได้ **575,061 sessions** และ **45 event templates** (Loghub มี 29)
- Sanity check กับ `Event_traces.csv`: session ที่**ความยาวตรงกับ Loghub 100.00%**, session ของ Loghub ที่หาไม่เจอ 0
- ความยาว session หลัง parse: median 19, mean 19.4

| ชุด | Sessions | Anomaly |
|-----|----------|---------|
| Train | 402,542 | 11,787 (2.93%) |
| Val | 86,259 | 2,525 (2.93%) |
| Test | 86,260 | 2,526 (2.93%) |

**Output:** `data/processed/HDFS/{train,val,test}.csv`, `checkpoints/drain3_hdfs.bin`

---

## 03 — Baseline Models (`03_baseline_models.ipynb`)

**Input:** `data/processed/HDFS/{train,val,test}.csv`

**สิ่งที่ทำ**
- แปลง sequence เป็น **Event Count Vector** ด้วย `CountVectorizer(token_pattern=r'\b\w+\b')` → matrix ขนาด (sessions, 45)
- **Logistic Regression** (Supervised): `class_weight='balanced'`, `max_iter=1000`, `random_state=42`
- **Isolation Forest** (Unsupervised): `n_estimators=100`, `contamination=0.0293`, `random_state=42`
- วัดผลด้วย classification report, confusion matrix, F1, PR-AUC แล้วบันทึกเป็น JSON

**Output:** `results/metrics/logistic_regression_hdfs.json`, `results/metrics/isolation_forest_hdfs.json`, `results/figures/05_confusion_matrix_lr.png`, `results/figures/06_confusion_matrix_iforest.png`

---

## 04 — Supervised Session-level LSTM (`04_lstm_deeplog.ipynb`)

**Input:** `data/processed/HDFS/{train,val,test}.csv`

**สิ่งที่ทำ**
- Vocabulary 47 tokens (`<PAD>`, `<UNK>` + 45 events)
- Dataset ระดับ session, padding ด้วย `pad_sequence` และใช้ `pack_padded_sequence` ในโมเดล
- โมเดล: Embedding(32) → LSTM 2 ชั้น (hidden 64, dropout 0.2) → Linear (59,937 parameters)
- Loss: `BCEWithLogitsLoss(pos_weight=33.1514)` (Normal ÷ Anomaly ใน train = 390,755 / 11,787)
- Optimizer: Adam, lr = 0.001, batch 256, 10 epochs, `SEED = 42`
- เลือก threshold จาก validation F1 (ไล่ 0.01–0.99) → **threshold = 0.95**, best validation F1 = 0.9880

**Output:** `checkpoints/lstm_deeplog_hdfs.pt`, `results/metrics/lstm_deeplog_hdfs.json`, `results/figures/05_lstm_training_loss.png`, `results/figures/06_lstm_confusion_matrix.png`

---

## 04b — DeepLog Next-Event Prediction (`04b_deeplog_next_event.ipynb`)

**Input:** `data/processed/HDFS/{train,val,test}.csv`

**สิ่งที่ทำ**
- Train ด้วย **Normal session เท่านั้น** — เมื่อไม่มี GPU โค้ดจะสุ่ม `MAX_TRAIN_SESSIONS = 25000` (รอบที่รันนี้รันบน CPU จึงใช้ **25,000 จาก 390,755** sessions)
- Sliding window `WINDOW_SIZE = 10` (pad ด้านหน้า) → 488,673 windows สำหรับทาย event ถัดไป
- โมเดล: Embedding(32) → LSTM 2 ชั้น (hidden 64, dropout 0.2) → Linear บน vocabulary 47 (62,927 parameters)
- Loss: `CrossEntropyLoss`, Adam lr = 0.001, batch 512, **5 epochs**, `SEED = 42`
- ตรวจจับ: event จริงไม่อยู่ใน Top-k → abnormal step; session ที่มี abnormal step ≥ 1 → Anomaly
- จูน k บน validation จาก `[1, 2, 3, 5, 7, 9, 12, 15]` → **best k = 5** (validation F1 0.6712)
- ท้าย notebook รวมตารางเปรียบเทียบจาก metrics JSON ของทุกโมเดล

**Output:** `checkpoints/deeplog_next_event_hdfs.pt`, `results/metrics/deeplog_next_event_hdfs.json`, `results/figures/07_deeplog_training_loss.png`, `results/figures/08_deeplog_val_k_tuning.png`, `results/figures/08_confusion_matrix_deeplog.png`

---

## ลำดับการรัน

```
01_eda  (ไม่จำเป็นต่อขั้นถัดไป)
02_log_parsing  →  สร้าง data/processed/HDFS/*.csv และ checkpoints/drain3_hdfs.bin
03 / 04 / 04b   →  อ่าน CSV จาก 02 (รันแยกกันได้)
```
ตารางเปรียบเทียบท้าย `04b` อ่าน metrics JSON ของ 03 และ 04 จึงต้องรัน 03 และ 04 ก่อน

## หมายเหตุ
- โมดูลใน `src/` (`data`, `models`, `training`, `utils`) ยังมีแค่ `__init__.py` — โค้ดทั้งหมดตอนนี้อยู่ใน notebook
