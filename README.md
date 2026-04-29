# EMG-Controlled 3D-Printed Prosthetic Hand (ESP32 + Myoelectric Actuators)

An EMG-controlled, modular **3D-printed prosthetic hand** using an **ESP32** microcontroller, **surface myoelectric actuators/sensors**, and **servo actuation** (fingers + optional wrist/elbow modules). The system reads EMG signals from forearm muscles, applies lightweight filtering and thresholding, and drives servo motors via a **PCA9685 PWM driver** to perform grasping motions.

> Based on the “Project Implementation Manual — EMG-Controlled 3D-Printed Prosthetic Hand with Myoelectric Actuators” (Group 3, 2025–26).

---

## Table of Contents
- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Bill of Materials (BOM)](#bill-of-materials-bom)
- [Mechanical Design & 3D Printing](#mechanical-design--3d-printing)
- [Electronics Wiring & Integration](#electronics-wiring--integration)
- [Software & Firmware Setup](#software--firmware-setup)
- [EMG Calibration Procedure](#emg-calibration-procedure)
- [Deployment Steps](#deployment-steps)
- [Configuration Reference](#configuration-reference)
- [Performance Evaluation & Testing](#performance-evaluation--testing)
- [Troubleshooting](#troubleshooting)
- [System Latency Model](#system-latency-model)
- [Future Enhancements](#future-enhancements)
- [Safety Notes](#safety-notes)
- [License](#license)
- [Acknowledgements](#acknowledgements)

---

## Project Overview

This project implements an **EMG-controlled prosthetic hand** where:
- **Myoelectric actuator/sensor module** captures surface EMG (sEMG) from residual limb muscles.
- **ESP32** samples the EMG via ADC and applies filtering + threshold logic.
- **PCA9685** generates stable PWM across multiple channels to drive servos.
- **Servo motors** actuate a tendon-driven mechanism for finger grasp + optional wrist/elbow modules.

---

## System Architecture

The system is structured in functional layers:

1. **Sensing Layer**
   - Surface EMG sensing module placed on forearm flexor region
   - Output: analog voltage proportional to muscle activation

2. **Processing Layer (ESP32)**
   - ADC sampling (noted in the doc using GPIO34 / ADC1_CH6)
   - Filtering: rolling average / EMA-like smoothing (doc mentions EMA coefficient update)
   - Thresholding: user-calibrated threshold with optional hysteresis/deadband behavior

3. **Actuation Layer**
   - PCA9685 16-channel PWM driver controls servo signals
   - Finger servos + optional wrist/elbow servos

4. **Power Layer**
   - 3S LiPo (11.1V)
   - LM2596 buck converters: ~6V servo rail and ~5V logic rail
   - Logic-level conversion for safe interfacing (as referenced)

---

## Bill of Materials (BOM)

Core components highlighted in the documentation:

- **Microcontroller:** ESP32 Dev Board (38-pin)
- **EMG Sensing:** Surface myoelectric actuator/sensor module (analog output)
- **PWM Driver:** PCA9685 (I2C, 16-channel)
- **Servos:**
  - MG90S micro servos (fingers)
  - MG996R high-torque servos (wrist / elbow / shoulder modules as applicable)
- **Power:**
  - 3S LiPo battery (11.1V)
  - LM2596 DC-DC buck converters (step-down)
  - Logic level converter (bi-directional)
- **Mechanical:** PLA+ filament, braided fishing line tendons, elastic orthodontic bands, M3/M4 fasteners, connectors/wiring

---

## Mechanical Design & 3D Printing

### Modular Assembly Structure
The prosthetic is designed as a **modular upper-limb skeleton** with replaceable modules, e.g.:
- Shoulder module (MG996R)
- Elbow module (MG996R)
- Wrist module (MG996R)
- Hand & finger module (multiple MG90S)

### 3D Printing Settings (as documented)
- **Material:** Generic PLA+
- **Nozzle temperature:** ~220°C
- **Bed temperature:** ~60°C
- **Layer height:** ~0.2 mm
- **Infill density:** ~40% (Gyroid)
- **Total print plates:** ~13
- **Total print time:** ~38 hours
- **Dimensional tolerance:** ±0.3 mm on critical mating surfaces

### Post-Processing & Assembly
- Sand/deburr critical mating surfaces
- Clear conduit channels (for tendons/wiring)
- Insert M3/M4 hardware (heat-set inserts or threaded inserts as applicable)
- Route braided fishing line through finger channels
- Attach elastic bands for finger extension return
- Verify open/close range manually before installing servos

---

## Electronics Wiring & Integration

### EMG Sensor → ESP32 (as shown)
- EMG analog **SIG** → **ESP32 GPIO34 (ADC1_CH6)**
- Sensor **VCC** → 5V logic rail
- Sensor **GND** → common ground

Notes:
- Ensure sensor output does **not exceed ADC-safe voltage**
- If needed, add a voltage divider inline (the doc references divider values as an option)

### PCA9685 → ESP32 (I2C)
- **SDA** → ESP32 GPIO21
- **SCL** → ESP32 GPIO22
- **VCC (logic)** → 5V logic rail
- **GND** → common ground
- **V+ (servo power)** → 6V servo rail (from buck converter)

### Servo Channel Mapping
A typical mapping used in the documentation:
- Ch 0: Thumb + Index (MG90S)
- Ch 1: Middle (reversed mounting, firmware compensates)
- Ch 2: Ring + Pinky
- Ch 3: Wrist (MG996R) neutral around STOP_VAL
- Ch 4: Elbow (MG996R) neutral around STOP_VAL

---

## Software & Firmware Setup

### Development Environment
- **Arduino IDE**
- ESP32 board package installed
- Serial monitor at **115200 baud**

### Firmware Capabilities (as documented)
- Reads EMG ADC
- Applies EMA/rolling filter (configurable coefficient)
- Compares against a user threshold
- Drives servos via PCA9685
- Hosts a **Wi‑Fi AP + dashboard**:
  - AP SSID shown: **Bionic-Arm-Pro**
  - Default dashboard URL shown: `http://192.168.4.1`

### Libraries Mentioned
- `WiFi.h` (ESP32 softAP)
- `WebServer.h` (HTTP server + endpoints)
- `Wire.h` (I2C)
- `Adafruit_PWMServoDriver.h` (PCA9685 PWM control)

---

## EMG Calibration Procedure

Calibration is required when sensor position changes:

1. Place the EMG sensor on the **forearm flexor** muscle belly; align electrodes properly.
2. Power on and open Serial Plotter at **115200**.
3. Observe:
   - `EMG_Signal` (filtered ADC)
   - `Threshold` (user threshold)
4. Relax arm completely; note resting mean ADC.
5. Flex wrist strongly; observe peak ADC.
6. Set `userThreshold` approximately halfway between rest mean and contraction mean.
7. Confirm with ~20 supervised grip trials:
   - Aim: minimal false positives at rest and reliable triggers on contraction
8. If false positives occur, raise threshold gradually.

---

## Deployment Steps

1. Flash ESP32 firmware via Arduino IDE.
2. Confirm EMG readings + threshold in Serial Monitor/Plotter.
3. Tune buck converters:
   - 6.0V servo rail
   - 5.0V logic rail
4. Connect PCA9685 and confirm I2C wiring (GPIO21/22).
5. Mount ESP32 + PCA9685 + buck converters into the forearm housing (use cable channels).
6. Connect 3S LiPo via XT60 and inline power switch.
7. Join Wi‑Fi AP (SSID: **Bionic-Arm-Pro**) and open dashboard `http://192.168.4.1`.
8. Attach EMG sensor to forearm and run calibration.
9. Enable EMG control; perform supervised grip trials.
10. Mount prosthetic on test rig; verify full ROM and safe operation.

---

## Configuration Reference

Key values (as documented):

- **EMG ADC Pin:** GPIO34
- **EMG Threshold (`userThreshold`):** default shown ~800 (adjust after calibration)
- **EMA Filter Coefficient:** 0.9 (retain) / 0.1 (new)
- **Wi‑Fi SSID:** Bionic-Arm-Pro
- **Wi‑Fi Password:** password123
- **Dashboard URL:** `http://192.168.4.1`
- **PWM Frequency:** 50 Hz
- **Wrist/Elbow Neutral:** STOP_VAL (~307 ticks shown)
- **Finger open/close pulse limits:** F_MIN / F_MAX / THUMB_MAX (as documented)
- **I2C pins:** SDA=GPIO21, SCL=GPIO22
- **Serial baud:** 115200
- **Servo rail voltage:** 6.0V
- **Logic rail voltage:** 5.0V

---

## Performance Evaluation & Testing

The documentation shows a test protocol and target metrics:

- **Grip Response Time:** target 55–180 ms (observed ~120–180 ms)
- **Control Accuracy:** target ≥90% (achieved ~90% in 20 trials)
- **EMG Signal Reliability (SNR):** ~15–20 dB observed

Also shown (mechanical/system):
- Finger extension return time (elastic bands): <200 ms consistently observed
- Wrist/elbow range test: full ±90° sweep in ~200–250 ms
- Structural integrity: no delamination under simulated grip forces
- Repeatability: consistent across 100+ grip trials with no tendon slippage

---

## Troubleshooting

- **Servo does not move on contraction**
  - Cause: threshold too high / EMG too low
  - Fix: recalibrate threshold; reposition sensor closer to muscle belly

- **False triggers at rest**
  - Cause: threshold too low / motion artifact noise
  - Fix: increase threshold; add hysteresis; check cable shielding

- **EMG ADC reads 0 or constant**
  - Cause: sensor power / ADC pin issue
  - Fix: verify 5V rail; check GPIO34; check wiring

- **Finger does not extend after grip**
  - Cause: elastic band broken / insufficient tension
  - Fix: replace/add bands; check dorsal anchor points

- **Servo jitter / oscillation**
  - Cause: PWM noise / insufficient supply
  - Fix: verify 6V rail under load; add bulk capacitor (~470µF suggested)

- **ESP32 resets under load**
  - Cause: brownout from LiPo sag
  - Fix: improve 5V stability; check buck converter tuning; add bulk capacitance

- **Tendon slips / incomplete close**
  - Cause: tendon anchor slip / spool winding
  - Fix: re-knot tendon; apply threadlock; ensure proper winding

- **Servo overheats**
  - Cause: stall current / wrong voltage
  - Fix: verify servo rail ≤6.2V; reduce motion range; avoid stall positions

- **Joint binds mechanically**
  - Cause: misalignment / insufficient clearance
  - Fix: sand mating surfaces; check insert alignment; reprint if needed

---

## System Latency Model

The manual models:
- `L_total = L_emg + L_processing + L_servo`

Typical values shown:
- `L_emg` (sensor conditioning): <5 ms
- `L_processing` (ESP32 ADC + rolling avg): <2 ms at 500 Hz sampling
- `L_servo` (mechanical response): ~120–180 ms (dominant)
- Total estimated: ~127–187 ms

---

## Future Enhancements

- ML-based gesture classification on ESP32
- Multiple grip patterns without hardware changes
- Add force sensors for closed-loop grip force control
- Vibrotactile haptic feedback using mini motors
- Stronger/flexible materials (PETG/TPU) for socket durability/comfort
- Testing with volunteers + clinical collaboration
- Add IMU for improved context awareness
- OTA firmware updates via ESP32 Wi‑Fi

---

## Safety Notes

- Always test with the prosthetic mounted on a **test rig** before wearing/handling.
- Keep **tendons and wiring** separated to prevent tangling and sudden snags.
- Verify **servo rail voltage** and avoid stall conditions to prevent overheating.
- Use an **inline power switch** and consider a fuse for battery safety.

---

## License

TBD

---

## Acknowledgements

See `CREDITS.md`.
