# HandsOn Day 6 - IMU Smooth Data Collection & Classification Benchmark

**Repository:** http://github.com/venki666/embai  
**Goal:** Collect stable IMU data for slow-moving actions and benchmark classification models.

---

## 1) Overview
This project builds an end-to-end pipeline:

**M5 IMU data collection (smooth logging) → CSV dataset → feature extraction → classification benchmark**

**Models evaluated**
- Decision Tree
- Random Forest
- SVM (RBF)
- MLP Neural Network
- MLP-BP Neural Network (epoch loop + early stopping)

---

## 2) Data Collection (IMU_SD_ST_AGE.ino)
### Sampling & Logging
- **Device:** M5 (built-in IMU) + SD card
- **Sampling rate:** 100 Hz
- **Record control:** Button A toggles logging ON/OFF
- **File naming:** `/imu_smooth_<millis>.csv` (unique per action)

### Smoothing (EMA Low-pass)
To reduce noise, EMA smoothing was applied:
- `ALPHA = 0.2`
- Logged signals include smoothed IMU channels + Madgwick roll/pitch/yaw.

### CSV Format
`timestamp, ax_smooth, ay_smooth, az_smooth, gx_smooth, gy_smooth, gz_smooth, roll, pitch, yaw`

---

## 3) Dataset
### Classes (5 actions)
- `m0_updown`
- `m1_leftright_small`
- `m2_leftright_speedup`
- `m3_rotate_slow`
- `m4_drinkwater`

### Windowing + Features (used for DT/RF/SVM/MLP-BP)
- **FS:** 100 Hz  
- **Window:** 100 samples (1 sec)  
- **Stride:** 50 samples (0.5 sec overlap)

**Feature extraction (45-D per window)**
- Channels (9): `ax, ay, az, gx, gy, gz, roll, pitch, yaw`
- Stats (5): `mean, std, min, max, msq(mean-square)`
- Total: 9 × 5 = **45 features/window**

### Dataset summary
- Total windows: **296**
- Class balance: ~59–60 windows per class
- Split (stratified): **60% train / 20% val / 20% test**
  - Train 177 / Val 59 / Test 60

---

## 4) Results (Test Set)
| Model | Accuracy | Precision | Recall | F1 | Notes |
|---|---:|---:|---:|---:|---|
| Decision Tree | 0.80 | 0.70 | 0.80 | 0.733 | `m0_updown → m2_leftright_speedup` confusion |
| Random Forest | 1.00 | 1.00 | 1.00 | 1.00 | Best params: n_estimators=50, max_depth=5 |
| SVM (RBF) | 1.00 | 1.00 | 1.00 | 1.00 | Best params: C=10, gamma=0.01 |
| MLP NN | 1.00 | 1.00 | 1.00 | 1.00 | Input=9 channels (no window stats) |
| MLP-BP | 1.00 | 1.00 | 1.00 | 1.00 | 45-D window stats + early stopping |

**Key observation**
- Decision Tree struggled to separate `m0_updown` vs `m2_leftright_speedup` with shallow depth.
- Random Forest, SVM(RBF), and MLP-based models achieved perfect separation for this dataset.
- Orientation channels (roll/pitch/yaw) and their statistics strongly contributed to class separability.

---

