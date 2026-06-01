# Wi-Fi CSI Room Occupancy Detection

> Privacy-preserving passive RF sensing system for room occupancy detection using Wi-Fi Channel State Information (CSI) — built on ESP32-C6 hardware. No camera, no identity, no data leaves the device.


## System Overview

This system uses ESP32-C6 units in a bistatic configuration (TX and RX placed on opposite sides of a room) to continuously monitor Wi-Fi signal propagation. When a person enters or occupies the sensing area between TX and RX, multipath propagation changes are captured as variations in CSI amplitude and packet delivery rate. An on-device ML classifier then determines occupancy state in real time.

**Current output states:** `NO HUMAN` / `HUMAN DETECTED`

**Target output states:** `VACANT` / `STATIC` / `ACTIVE`

### Privacy by Design

All inference runs on-device (SBC). Raw CSI data is never transmitted to external servers. No camera, no microphone, no biometric data.

## Current Performance

Tested in a controlled 3×3 m environment, single TX–RX pair:

- Dataset: 7,000 samples (7 categories × 1,000 samples)
- Classifier: Random Forest with delta-CSI features
- Accuracy: 97% (cross-validation mean 98% ±2.7%)
- Real-time latency: ~3 seconds (30-window majority voting)
- False positive rate: 0/15 | False negative rate: 1/16


## Hardware Stack

| Component | Role |
|-----------|------|
| ESP32-C6 (TX) | CSI transmitter — Station mode |
| ESP32-C6 (RX) | CSI receiver — SoftAP mode |
| USB-Serial | Data streaming to host PC |

## Deployment Geometry

TX and RX must be placed on **opposite sides** of the monitored area. The system operates on bistatic sensing principles — detection sensitivity is highest when the target is within the Fresnel zone between TX and RX.

## Software Pipeline

| Script | Function |
|--------|----------|
| `csi_logger.py` | Log raw CSI data from RX unit via serial |
| `csi_calibrate.py` | Baseline calibration for empty-room reference |
| `csi_train.py` | Train ML classifier on collected CSI dataset |
| `csi_realtime.py` | Real-time inference and presence detection |
| `csi_visualizer.py` | CSI amplitude visualization and analysis |

## Procedure

**1. Flash firmware**

Using Arduino IDE:
- Open `Firmware/TX/TX.ino` → select ESP32-C6 board → upload to transmitter unit
- Open `Firmware/RX/RX.ino` → select ESP32-C6 board → upload to receiver unit

**2. Installing dependencies**
```bash
pip install -r requirements.txt
```

**3. Calibration**
```bash
python csi_calibrate.py
```

**4. Model training and data collection**
```bash
python csi_logger.py
python csi_train.py
```

**5. Run real-time detection**
```bash
python csi_realtime.py
```

## Realtime Serial Monitor



https://github.com/user-attachments/assets/d93277cf-716b-4350-8b9f-c59040b73f4f



## Confusion Matrix

<img width="933" height="864" alt="Screenshot 2026-05-17 102632" src="https://github.com/user-attachments/assets/fcd48796-21b0-4753-a820-7e0dd73698d0" />

<img width="1276" height="789" alt="Screenshot 2026-05-17 102648" src="https://github.com/user-attachments/assets/9e6f02f5-90af-4155-8a08-48de7dbfdcba" />

## Dataset

Sample CSI data is provided in `data/sample/csi_sample.csv`.
Full dataset available on request.

## Project Status

- [x] CSI data logging via pyserial
- [x] ML classifier training (Random Forest, 97% accuracy)
- [x] Baseline calibration (manual trigger, 300 samples) — to be superseded by zero-touch auto-baseline
- [x] Real-time detection in controlled environment
- [ ] Packet rate as complementary sensing feature (hypothesis stage)
- [ ] Calibration-free auto-baseline via GMM/PCA (zero human intervention)
- [ ] Adaptive threshold per subcarrier (μ + k·σ)
- [ ] Continuous self-update via Exponential Moving Average
- [ ] Activity state output: `VACANT` / `STATIC` / `ACTIVE`
- [ ] Occupancy count estimation (1 person vs multiple)

## Known Limitations

- System tested in a 3×3 meter room under controlled conditions
- Detection consistency varies with environmental changes (furniture repositioning, temperature, interference)
- Classifier requires recalibration when deployed in a new environment

## License

MIT License — see [LICENSE](LICENSE) for details.
