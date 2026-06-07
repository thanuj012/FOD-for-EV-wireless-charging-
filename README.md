# FOD System for EV Wireless Charging using LTspice and MATLAB

> Foreign Object Detection (FOD) for Electric Vehicle Dynamic Wireless Power Transfer (EV DWPT) systems using coupled inductor modeling in LTspice and Wavelet Transform analysis in MATLAB.

---

## Overview

Wireless Power Transfer (WPT) systems for EVs are vulnerable to foreign metallic objects placed between the transmitter and receiver coils. These objects absorb energy via eddy currents, causing heat buildup and efficiency loss — posing a serious safety and reliability risk.

This project simulates a complete SS (Series-Series) topology WPT system in LTspice, models foreign objects as magnetically coupled inductors with eddy current resistance, and uses Wavelet Transform in MATLAB to extract features that distinguish normal operation from FOD conditions.

---

## System Architecture

```
DC Source (50V)
      ↓
H-Bridge Inverter (4x NMOS, 15kHz switching)
      ↓
Series Resonant Tank (C3 + L1, primary)
      ↓ magnetic coupling (K = 0.3–0.7)
Series Resonant Tank (L2 + C4, secondary)
      ↓
Load Resistor R2
```

FOD object modeled as:
```
L3 (eddy current inductance) + R_fod (eddy current loss)
      ↓ weak magnetic coupling to L1 (K2 = 0.1–0.3)
```

---

## Circuit Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Supply Voltage (Vcc) | 50V | DC input |
| Switching Frequency | 15 kHz | H-bridge gate signals |
| Primary Inductance (L1) | 100 µH | Transmitter coil |
| Secondary Inductance (L2) | 100 µH | Receiver coil |
| Resonant Capacitors (C3, C4) | 1.126 µF | Tuned to 15 kHz |
| Coupling Coefficient (K) | 0.3–0.69 | TX-RX coil coupling |
| Load Resistance (R2) | 10 Ω | Secondary load |
| Gate Drive Voltage | 30V | NMOS gate signal |
| MOSFET Model | MYMOS (Level-1 NMOS) | VTO=2V, KP=100 |

### Resonant Frequency Verification

$$f_r = \frac{1}{2\pi\sqrt{LC}} = \frac{1}{2\pi\sqrt{100\mu \times 1.126\mu}} \approx 15\ \text{kHz}$$

---

## FOD Modeling

Foreign metallic objects are modeled as a third magnetically coupled inductor loop:

| Object Type | L3 | R_fod | K2 (coupling to L1) |
|-------------|-----|-------|----------------------|
| Small metal (bolt) | 10 µH | 1 Ω | 0.1 |
| Large metal (sheet) | 100 µH | 0.1 Ω | 0.3 |
| Copper object | 50 µH | 0.01 Ω | 0.2 |

The eddy current loop (L3 + R_fod shorted) magnetically loads the primary coil L1, causing measurable changes in I(L1).

---

## MATLAB Signal Processing Pipeline

```
LTspice Export (I(L1) waveform as .txt)
          ↓
Load into MATLAB
          ↓
Wavelet Decomposition (db4, Level 5)
          ↓
Feature Extraction:
  - RMS Current
  - Peak Current
  - Wavelet Energy per Level (L1–L5)
          ↓
Comparison: Normal vs FOD
          ↓
Threshold-based Detection
```

### Wavelet Features Extracted

| Feature | Normal | FOD Present | Change |
|---------|--------|-------------|--------|
| RMS Current | 3.21 A | 3.38 A | +5.64% |
| Peak Current | 13.86 A | 16.51 A | +23.64% |
| Wavelet Energy L1 | 383.75 | 583.40 | +52.03% |
| Wavelet Energy L2 | 5279.37 | 7209.68 | +36.56% |
| Wavelet Energy L3 | 15811.17 | 21709.45 | +37.30% |

A FOD event causes a significant **increase in wavelet energy across all decomposition levels**, particularly at levels 1–3, which correspond to high-frequency disturbance components introduced by the eddy current interaction.

---

## Key Results

- Primary coil current I(L1) shows clean sinusoidal resonance at 15 kHz under normal conditions
- FOD object causes measurable increase in wavelet energy (up to 52% at Level 1)
- Peak current increases by 23.64% when FOD is present
- Wavelet-based features provide clear separation between normal and FOD conditions

---

## Project Structure

```
Wapati/
├── SYSTEM 1 SS H BRIDGE.asc     # LTspice schematic
├── SYSTEM 1 SS H BRIDGE.net     # LTspice netlist
├── normal.txt                    # Exported I(L1) - no FOD
├── fod_steel.txt                 # Exported I(L1) - FOD present
├── FOD_DETECTION.m               # MATLAB analysis script
└── README.md
```

---

## How to Run

### LTspice Simulation
1. Open `SYSTEM 1 SS H BRIDGE.asc` in LTspice
2. Run transient simulation: `.tran 1u 10m`
3. Plot I(L1) → right click → File → Export as text → `normal.txt`
4. Add L3 + R_fod in parallel with L1 with coupling directive `K2 L1 L3 0.1`
5. Re-run and export as `fod_steel.txt`

### MATLAB Analysis
1. Set MATLAB working directory to project folder
2. Run `FOD_DETECTION.m`
3. Check command window for RMS, peak, and wavelet energy comparison
4. View 3-panel figure for visual comparison

---

## Tools Used

| Tool | Purpose |
|------|---------|
| LTspice XVII / 24 | Circuit simulation |
| MATLAB R20xx | Signal processing and wavelet analysis |
| Wavelet Toolbox (MATLAB) | `wavedec`, `detcoef` functions |

---

## Future Work

- Add full-bridge rectifier on secondary side for DC output
- Reduce coupling coefficient to realistic EV WPT range (K = 0.1–0.3)
- Simulate multiple FOD scenarios and train SVM classifier
- Implement real-time threshold detection logic
- Validate against hardware prototype

---

## Author

**Thanuj Srinivassalou**
B.E. Electrical and Electronics Engineering
SSN College of Engineering, Chennai
GitHub: [github.com/thanuj012](https://github.com/thanuj012)

---

## References

1. Soft Fault Diagnosis of WPT System Main Circuit using Wavelet Transform and SVM
2. SAE J2954 Wireless Power Transfer Standard for EVs
3. Wireless Power Transfer for Electric Vehicles — IEEE Transactions on Power Electronics
