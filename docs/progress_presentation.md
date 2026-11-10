---
marp: true
theme: default
paginate: true
header: "LogSentinel — Progress Presentation | CS461 Deep Learning"
footer: "ภาควิชาวิทยาการคอมพิวเตอร์และสารสนเทศ มหาวิทยาลัยกรุงเทพ"
style: |
  section {
    font-family: 'Sarabun', 'Segoe UI', Tahoma, sans-serif;
    font-size: 22px;
    padding: 40px;
  }
  h1 { color: #1a237e; font-size: 36px; }
  h2 { color: #0d47a1; font-size: 28px; border-bottom: 2px solid #1976d2; padding-bottom: 8px; }
  h3 { color: #283593; font-size: 24px; }
  table { font-size: 18px; }
  .highlight { color: #d32f2f; font-weight: bold; }
  .success { color: #2e7d32; font-weight: bold; }
  .badge { background: #e3f2fd; color: #0d47a1; padding: 4px 8px; border-radius: 4px; font-size: 16px; }
  .note { background: #fff9c4; padding: 10px; border-left: 5px solid #fbc02d; font-size: 16px; margin-top: 10px; }
---

# LogSentinel 🛡️
### ระบบตรวจจับความผิดปกติใน System Log ด้วย Deep Learning สำหรับศูนย์ปฏิบัติการความปลอดภัยไซเบอร์ (SOC)
**Deep Learning-based Log Anomaly Detection for Security Operations Centers**

---

### 👥 สมาชิกในกลุ่ม (Team Members)
- **ธนกร จำนงประโคน** (1660701168)
- **ศิวกร สุภาวงค์** (1660700251)
- **นิธิสิทธิ์ เสนาหาญ** (1660701317)
- **ภูวดล ผิวบางกูล** (1660702067)

**วิชา:** CS461 Deep Learning — Final Project Progress Update  
**อาจารย์ที่ปรึกษา:** อาจารย์ประจำวิชา CS461

---

## สารบัญการนำเสนอ (Agenda)

1. **ที่มาและความสำคัญ** (Problem Statement & SOC Challenges)
2. **ภาพรวมระบบ** (System Architecture Pipeline)
3. **การสำรวจข้อมูล** (EDA & Accuracy Paradox)
4. **การแปลงและจัดกลุ่ม Log** (Drain3, Sessionization & Stratified Split)
5. **โมเดลพื้นฐาน** (Baseline: Logistic Regression & Isolation Forest)
6. **Supervised LSTM** (Session-level Sequence Classifier)
7. **DeepLog** (Next-Event Prediction, Normal-only)
8. **สรุปผลและเปรียบเทียบทุกโมเดล** (Comparative Evaluation)
9. **งานถัดไป** (Transformer, BGL & Prototype)

---

## 1. ที่มาและความสำคัญในศูนย์ SOC

### 🚨 ปัญหาจริงของศูนย์เฝ้าระวังความปลอดภัย (SOC Challenges):
- **ปริมาณ Log มหาศาล:** ระบบ Hadoop (HDFS) ผลิต Log กว่า **11.17 ล้านบรรทัด ภายในเวลาไม่ถึง 2 วัน** เกินกว่าที่นักวิเคราะห์จะอ่านด้วยมือ
- **ความผิดปกติมีน้อยมาก (~2.9%):** ถูกกลบอยู่ในกิจกรรมปกติของระบบ
- **ข้อจำกัดของ Rule-based / Regex แบบเดิม** (เทียบได้กับ Signature ของ Snort/YARA):
  - ต้องเขียนและอัปเดตกฎด้วยมือตลอดเวลา
  - จับได้เฉพาะรูปแบบที่รู้จัก ความผิดปกติรูปแบบใหม่ (Novel Anomalies) หลุดรอด
  - ก่อให้เกิด **Alert Fatigue** (แจ้งเตือนผิดบ่อยจนนักวิเคราะห์เพิกเฉย)

### 🎯 เป้าหมายของโครงงาน:
- พัฒนาระบบที่ **เรียนรู้รูปแบบพฤติกรรมของ Log ได้เอง**
- จำแนกความผิดปกติระดับ Session ด้วย **F1-Score $\ge 80\%$** บนชุดข้อมูล HDFS
- สร้าง Web Application Prototype สำหรับทีม SOC

---

## 2. ภาพรวมสถาปัตยกรรมการทำงาน (Pipeline)

```
[1. Raw Logs]       → HDFS.log (11.17 ล้านบรรทัด)
       ↓
[2. Log Parsing]    → Drain3 Parser + Masking (<BLK>, <IP>, <NUM>)
       ↓
[3. Sessionizing]   → รวมเหตุการณ์ตาม block_id
       ↓
[4. Stratified]     → แบ่ง 70% Train / 15% Val / 15% Test (ระดับ Session)
       ↓
[5. Modeling]       → Baseline (Event Count) vs LSTM / DeepLog (Sequence)
       ↓
[6. SOC Metrics]    → ประเมินด้วย F1-Score, Recall, Precision, PR-AUC
```

> **หลักคิดสำคัญ:** เปลี่ยนข้อความ Log ที่คอมพิวเตอร์ไม่เข้าใจ ให้เป็น **ลำดับเหตุการณ์เชิงตัวเลข (Sequence of Event IDs)** ที่ป้อนให้โมเดลได้

---

## 3. การสำรวจข้อมูล (EDA)

### 🔍 สิ่งที่พบจากไฟล์ HDFS_v1 (11,175,629 บรรทัด):
- **โครงสร้าง Log:** `<Date> <Time> <Pid> <Level> <Component>: <Content>`
- **Log Level Distribution:**
  - `INFO`: **>95%** (การทำงานปกติทั่วไป)
  - `WARN` / `ERROR`: **<1%**
  - 💡 **ข้อคิดทาง Security:** หากใช้ Rule ดักจับแค่คำว่า `ERROR` จะพลาดความผิดปกติแบบ **Silent Failure** ทั้งหมด
- **สัดส่วนคลาส (Class Imbalance):**
  - Session ทั้งหมด: **575,061 Sessions**
  - Normal: **558,223 Sessions (97.07%)**
  - Anomaly: **16,838 Sessions (2.93%)**

### 🧠 สิ่งที่ EDA บอกเราสำหรับการออกแบบขั้นต่อไป
- ข้อมูลไม่สมดุลมาก → ต้องใช้ **Class Weighting** และ **Stratified Split**
- ความยาว Session ไม่เท่ากัน → โมเดลลำดับต้องใช้ **Padding / Packing**

---

## 3. ความย้อนแย้งของความแม่นยำ (Accuracy Paradox)

### ⚠️ ทำไมโครงงานนี้ไม่ใช้ "Accuracy" เป็นเกณฑ์ตัดสิน?

สมมติสร้างโมเดลที่ **ทายว่า "Normal" ทุกกรณี**:
- **Accuracy สูงถึง 97.07%** (เพราะข้อมูลส่วนใหญ่ปกติ)
- แต่ในมุม SOC: ความผิดปกติ 16,838 ครั้ง **หลุดรอดทั้งหมด (Recall = 0%)**
- เหมือน IDS ที่ไม่เคยแจ้งเตือนเลย แต่รายงานว่า "แม่นยำ 97%"

### 🛡️ เกณฑ์การวัดผลที่ใช้ (ตามที่เรียนเรื่อง Evaluation Metrics):
1. **Recall** $= \frac{TP}{TP+FN}$: เกิดเหตุจริง 100 ครั้ง จับได้กี่ครั้ง (ป้องกันการหลุดรอด)
2. **Precision** $= \frac{TP}{TP+FP}$: เตือน 100 ครั้ง เป็นเหตุจริงกี่ครั้ง (ลด Alert Fatigue)
3. **F1-Score** (ตัวชี้วัดหลัก): Harmonic Mean ของ Precision และ Recall
4. **PR-AUC**: วัดคุณภาพทุก Threshold — เหมาะกับข้อมูลไม่สมดุลมากกว่า ROC-AUC เพราะไม่ถูกจำนวน True Negative มหาศาลดันให้ดูดี

---

## 4. การแปลง Log ด้วย Drain3 (Log Parsing)

### ❓ ทำไมต้อง Parse? ทำไมไม่ป้อนข้อความดิบ?
- **ปัญหา Vocabulary ไม่จำกัด:** Log มีค่าตัวแปรเปลี่ยนตลอด (IP, Port, Block ID) ถ้าไม่สกัดออก โมเดลจะเจอ "คำใหม่" เป็นล้านคำ
- เทียบกับ SIEM: คือขั้นตอน **Normalization** ก่อนเขียน Correlation Rule

### 🛠️ เทคนิคที่ใช้: Drain3 (Fixed-depth Tree Parser)
- จัดกลุ่มประโยคตามความยาวและ Token แรก ๆ แล้ววัดความเหมือนกับ Template ที่มีอยู่
- เพิ่ม **Regex Masking** ก่อน Parse:
  - `blk_-?\d+` $\rightarrow$ `<BLK>`
  - `\d+\.\d+\.\d+\.\d+(:\d+)?` $\rightarrow$ `<IP>`
  - `\b\d+\b` $\rightarrow$ `<NUM>`
- **เหตุผลที่เลือก Drain3:** ทำงานแบบ Online (ทีละบรรทัด) และเร็วพอสำหรับ 11 ล้านบรรทัด ซึ่งจำเป็นต่อการใช้ซ้ำใน Web App

### ✅ ผลลัพธ์
- เหลือ Event Templates เพียง **45 รูปแบบ ($E_1 \dots E_{45}$)** ตรงกับ Reference ของ Loghub **100%**

---

## 4. Sessionization & การแบ่งข้อมูล

### 1. รวมกลุ่มตาม Session (`block_id`)
- **เหตุผล:** ในระบบกระจาย เหตุการณ์ของบล็อกเดียวกันกระจายสลับกับบล็อกอื่น
- การรวมตาม `block_id` ทำให้โมเดลเห็นลำดับตั้งแต่ Allocate $\rightarrow$ Write $\rightarrow$ Replicate $\rightarrow$ Close
- Label อยู่ระดับ `block_id` (จาก `anomaly_label.csv`) จึงเป็นหน่วยที่ใช้จำแนกได้ตรงที่สุด

### 2. Stratified Session-Level Split (70% / 15% / 15%)
- **แบ่งที่ระดับ Session:** ไม่ตัดกลาง Session เพื่อป้องกัน **Data Leakage** (โมเดลเห็นข้อสอบตอนเรียน)
- **Stratified:** คงสัดส่วน Anomaly **2.93%** ในทุกชุด
- **ทำไมต้องมี Validation Set:** ใช้เลือก Hyperparameter / Threshold โดยไม่แตะ Test Set — Test ใช้วัดผลครั้งสุดท้ายเท่านั้น

| ชุดข้อมูล | Sessions | Anomaly |
|---|---|---|
| Train | 402,542 | 11,787 |
| Validation | 86,259 | 2,525 |
| Test | 86,260 | 2,526 |

---

## 5. โมเดลพื้นฐาน (Baseline Models)

### 🎯 ทำไมต้องมี Baseline?
เพื่อเป็นเกณฑ์วัด (Benchmark) พิสูจน์ว่าโมเดล Deep Learning ที่ซับซ้อนกว่าคุ้มค่าจริง

### 🛠️ เทคนิคที่ใช้
- **Feature: Event Count Vector** (`CountVectorizer`) — นับความถี่ของ $E_1 \dots E_{45}$ ในแต่ละ Session ได้เวกเตอร์ขนาดคงที่ (แนวคิดเดียวกับ Bag-of-Words)
- **Logistic Regression (Supervised):** Linear Classifier + Sigmoid ใช้ `class_weight='balanced'` เพื่อเพิ่มน้ำหนัก Loss ของคลาส Anomaly ที่มีน้อย
- **Isolation Forest (Unsupervised):** สุ่มแบ่งข้อมูลด้วยต้นไม้ จุดที่ถูก "แยกออก" ได้เร็ว (Path สั้น) คือจุดผิดปกติ ตั้ง `contamination = 0.0293` ตามสัดส่วน Anomaly จริง

| โมเดล | การเรียนรู้ | Precision | Recall | F1-Score | PR-AUC |
|---|---|---|---|---|---|
| **Logistic Regression** | Supervised | **96.12%** | **100.00%** | **98.02%** | **97.53%** |
| **Isolation Forest** | Unsupervised | 64.74% | 62.67% | **63.69%** | 73.36% |

---

## 5. ข้อจำกัดของ Baseline

### 🔍 Logistic Regression ได้ 98% แล้ว ทำไมยังต้องไปต่อ?
- **มองไม่เห็นลำดับ (Blind to Order):** Event Count Vector รู้แค่ว่ามี Event อะไร กี่ครั้ง แต่ **ไม่รู้ลำดับก่อน-หลัง**
- **ตัวอย่าง:**
  - ปกติ: `เสียบบัตร ATM` $\rightarrow$ `ใส่รหัส` $\rightarrow$ `ถอนเงิน`
  - ผิดปกติ: `ถอนเงิน` $\rightarrow$ `เสียบบัตร ATM` $\rightarrow$ `ใส่รหัส`
  - ทั้งสองมีจำนวน Event เท่ากัน → Baseline มองว่าเหมือนกัน
- **ต้องใช้ Label ครบ:** LR เรียนจากตัวอย่าง Anomaly ที่มีเฉลย คล้าย Signature-based ที่รู้จักเฉพาะสิ่งที่เคยเห็น
- **Isolation Forest** ไม่ใช้ Label แต่ F1 ต่ำกว่าเป้า (63.69%)

> จึงเป็นเหตุผลที่ต้องใช้ **โมเดลลำดับ (RNN/LSTM, Transformer)** ที่เข้าใจบริบทตามเวลา

---

## 6. Supervised LSTM — แนวคิดและเหตุผล

### 🧠 ทำไมเลือก LSTM?
- **RNN** อ่านลำดับทีละสเต็ปและจำบริบทใน Hidden State แต่ลำดับยาวจะเกิด **Vanishing Gradient**
- **LSTM** แก้ด้วย **Gate** 3 ตัว (Forget / Input / Output) และ **Cell State** ทำให้จำเหตุการณ์ที่เกิดไกลออกไปได้
- เทียบกับ SOC: เหมือนนักวิเคราะห์ที่อ่าน Timeline ของ Incident ตามลำดับ ไม่ใช่แค่นับจำนวน Alert

### 🛠️ สถาปัตยกรรม
```
Event IDs → Embedding(32) → 2-Layer LSTM(hidden 64, dropout 0.2) → Linear → Sigmoid → P(Anomaly)
```
- **Embedding:** แปลง Event ID เป็นเวกเตอร์ที่เรียนรู้ได้ แทน One-hot (Event ที่มักเกิดร่วมกันจะมีเวกเตอร์ใกล้กัน)
- **Padding + `pack_padded_sequence`:** รองรับ Session ยาวไม่เท่ากัน โดยไม่ให้ค่า Pad มีผลต่อการเรียนรู้
- **Dropout 0.2:** ลด Overfitting ระหว่างชั้น LSTM

---

## 6. Supervised LSTM — การฝึกและผลลัพธ์

### 🛠️ เทคนิคการฝึก
- **Loss: `BCEWithLogitsLoss(pos_weight=33.15)`** — Binary Cross-Entropy สำหรับ 2 คลาส, `pos_weight` = Normal ÷ Anomaly ใน Train เพื่อให้การพลาด Anomaly 1 ครั้ง "เจ็บ" เท่ากับพลาด Normal ~33 ครั้ง
- **Optimizer: Adam** (lr = 0.001), Batch 256, 10 Epochs
- **Threshold Tuning บน Validation:** ไม่ใช้ 0.5 ตายตัว แต่ไล่หาค่าที่ให้ F1 สูงสุดบน Validation ได้ **0.95** แล้วจึงนำไปใช้กับ Test

### ✅ ผลลัพธ์บน Test Set (86,260 sessions)
- **F1-Score: 98.71%** (Baseline LR: 98.02%) · Precision 97.64% · Recall 99.80% · PR-AUC 99.96%
- **False Alarm ลดลง:** จาก 102 เคสใน Logistic Regression **เหลือ 61 เคส**
- จับได้ **2,521 จาก 2,526 เคส** (หลุดรอด 5 เคส)

---

## 7. DeepLog — แนวคิดและเหตุผล

### Pure DeepLog (Du et al., ACM CCS 2017)
- **ปัญหาที่ต้องการแก้:** ในงานจริง ความผิดปกติใหม่ ๆ **ไม่มี Label** โมเดลจึงควรเรียนจาก **Log ปกติล้วน (Normal Only)**
- เทียบกับ SOC: คล้าย **Behavioral Baseline ของ UEBA** — เรียนว่า "ปกติ" หน้าตาเป็นอย่างไร อะไรที่หลุดจากนั้นคือสิ่งที่ต้องตรวจ

### 🛠️ เทคนิคที่ใช้
- **Sliding Window ($h = 10$):** ตัด Session ปกติเป็นหน้าต่างละ 10 Event ให้ทาย Event ถัดไป
- **Next-Event Prediction = Multi-class Classification:** LSTM (Embedding 32, Hidden 64, 2 ชั้น) → Linear → **Softmax** บน 47 คลาส ใช้ **`CrossEntropyLoss`**
- **Top-$k$ Detection:** ถ้า Event ที่เกิดจริงไม่อยู่ใน $k$ อันดับที่โมเดลทาย → Abnormal Step; Session ที่มี Abnormal Step $\ge 1$ → **Anomaly**
- **เลือก $k$ บน Validation:** ทดลอง $k \in \{1,2,3,5,7,9,12,15\}$ ได้ $k = 5$ ($k$ น้อย = เข้มงวด Recall สูง แต่แจ้งเตือนผิดมาก / $k$ มาก = ผ่อนปรน)

---

## 7. DeepLog — ผลลัพธ์และข้อจำกัด

### ✅ ผลลัพธ์บน Test Set (86,260 sessions, $k = 5$)

| Precision | Recall | F1-Score |
|---|---|---|
| **86.77%** | 56.85% | **68.69%** |

- จับได้ **1,436 จาก 2,526** Anomaly · แจ้งเตือนผิด **219** ครั้ง
- **Precision สูง:** เมื่อเตือนมักเป็นเหตุจริง แต่ **Recall ยังต่ำ** — ยังต่ำกว่าเป้า 80%

### ⚠️ ข้อจำกัดของการทดลองรอบนี้
- ฝึกบน CPU จึงสุ่มใช้ Normal Session เพียง **25,000 จาก 390,755** และฝึกแค่ **5 Epochs**
- โมเดลอาจยังเห็นรูปแบบปกติไม่ครบ → ทาย Event ถัดไปผิด หรือให้ Top-5 ไม่ครอบคลุมพอ

### 🔧 แผนปรับปรุง
- ฝึกด้วย Normal Session ทั้งหมดบน GPU · เพิ่ม Epoch · ทดลองขนาด Window / Hidden

---

## 8. สรุปผลการทดลองบน HDFS Test Set

ประเมินบน Test Set ชุดเดียวกันทั้งหมด (**86,260 Sessions**):

| โมเดล | เทคนิคหลัก | ข้อมูลที่ใช้ฝึก | Precision | Recall | F1-Score | PR-AUC |
|---|---|---|---|---|---|---|
| **Supervised LSTM** | Embedding + LSTM, Weighted BCE | Supervised | 97.64% | 99.80% | **98.71%** | 99.96% |
| **Logistic Regression** | Event Count + Class Weight | Supervised | 96.12% | 100.00% | **98.02%** | 97.53% |
| **DeepLog** | Sliding Window + Top-$k$ | Normal Only | 86.77% | 56.85% | **68.69%** | N/A |
| **Isolation Forest** | Random Tree Partition | Unsupervised | 64.74% | 62.67% | **63.69%** | 73.36% |

> 🏆 **โมเดลที่ใช้ Label (Supervised LSTM, Logistic Regression) ผ่านเป้า F1 $\ge 80\%$ แล้ว**
> ⚠️ **โมเดลที่ไม่ใช้ Label ของ Anomaly (DeepLog, Isolation Forest) ยังต่ำกว่าเป้า** — เป็นโจทย์หลักของงานถัดไป

---

## 8. สิ่งที่เราเรียนรู้

- **ข้อมูลไม่สมดุลต้องจัดการทุกขั้น:** Stratified Split → Class Weight / `pos_weight` → Threshold Tuning → วัดด้วย F1 / PR-AUC
- **Validation Set สำคัญ:** ทุกการตัดสินใจ (Threshold 0.95, $k = 5$) เลือกบน Validation เพื่อให้ผลบน Test เชื่อถือได้
- **Supervised ≈ Signature-based:** แม่นมากกับรูปแบบที่มีตัวอย่าง แต่ต้องพึ่ง Label
- **Normal-only ≈ Anomaly-based IDS:** ไม่ต้องมี Label ของการโจมตี แต่ยากกว่าและยังต้องปรับปรุง
- **บน HDFS โมเดลเรียบง่ายก็ได้ผลสูง:** Deep Learning จึงต้องพิสูจน์คุณค่าด้านอื่นด้วย เช่น การอธิบายผล (Explainability) และการใช้ข้ามระบบ (Generalization)

---

## 9. งานถัดไป

```
[1] Transformer Encoder
 → Multi-Head Self-Attention จับความสัมพันธ์ระยะไกล + Attention Heatmap อธิบายว่า Event ใดทำให้ถูกตัดสินว่าผิดปกติ
       ↓
[2] ปรับปรุง DeepLog
 → ฝึกด้วย Normal Session ทั้งหมดบน GPU และจูน Hyperparameter
       ↓
[3] เปรียบเทียบทุกโมเดล + ทดสอบข้ามระบบบน BGL
 → ทดสอบ Generalization กับ Log ซูเปอร์คอมพิวเตอร์ BGL (4.75 ล้านบรรทัด)
       ↓
[4] Web Application Prototype (Streamlit)
 → อัปโหลด Log, วิเคราะห์ราย Session และไฮไลต์ Event ที่น่าสงสัยให้ SOC Analyst
       ↓
[5] รายงานฉบับสมบูรณ์และการนำเสนอ
```

---

## สรุปสิ่งที่ส่งมอบแล้ว (Delivered Artifacts)

- ✅ **Notebooks:**
  - `notebooks/01_eda.ipynb` (EDA & Data Insights)
  - `notebooks/02_log_parsing.ipynb` (Drain3 Mining & Data Splits)
  - `notebooks/03_baseline_models.ipynb` (LR & Isolation Forest)
  - `notebooks/04_lstm_deeplog.ipynb` (Supervised Session LSTM)
  - `notebooks/04b_deeplog_next_event.ipynb` (DeepLog Next-Event)
- ✅ **Datasets & Checkpoints:**
  - `data/processed/HDFS/{train,val,test}.csv` (ขนาดรวม 51 MB)
  - `checkpoints/drain3_hdfs.bin`, `lstm_deeplog_hdfs.pt`, `deeplog_next_event_hdfs.pt`
- ✅ **Metrics & Figures:**
  - JSON Metrics ใน `results/metrics/` และกราฟ Confusion Matrix / Training Loss ใน `results/figures/`
- ✅ **AI Assistant Customization:**
  - `GEMINI.md`, `CLAUDE.md` และ `.claude/skills/logsentinel/SKILL.md`

---

# Q & A 💬
### ขอขอบคุณคณะกรรมการและอาจารย์ทุกท่าน
**ทีม LogSentinel พร้อมตอบทุกข้อซักถามครับ**
