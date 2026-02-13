# Smart BMS Viewer Dashboard (UAV Pack) — Display + Measurement Implementation

## 1) Purpose
The BMS Viewer Dashboard provides **real-time visibility**, **safety status**, and **state-estimation outputs** for ourbattery pack.

---

## 2) What the Dashboard Will Display

### 2.1 Summary Tiles (Top Row)
- **SOC (Kalman)**: `__ %`
- **SOC (Coulomb Count)**: `__ %`
- **Pack Voltage (Vpack)**: `__ V`
- **Pack Current (Ipack)**: `__ A` (signed; +charge / -discharge)
- **Pack Power**: `P = Vpack × Ipack` → `__ W`
- **Estimated Time Remaining** (optional): `__ min` (based on rolling average current)

### 2.2 Cell Monitoring (Critical for UAV)
- **Per-cell voltages**: `V1..Vn` (table + bar chart)
- **Vmax / Vmin**
- **Cell imbalance**: `ΔV = Vmax − Vmin`
- **Cell status indicators**:
  - OK
  - Warning (approaching limit)
  - Fault (limit exceeded)

### 2.3 Temperature Monitoring
- **Temperatures**: `T1..Tk` (typically: pack surface, near cells, near MOSFETs)
- **Tmax**
- Status indicators:
  - OK / Warning / Fault

### 2.4 Protection & Fault Panel (Safety)
Show a clear “traffic light” state for each protection:
- **Undervoltage (UV)**
- **Overvoltage (OV)**
- **Overcurrent discharge (OCD)**
- **Overcurrent charge (OCC)**
- **Short-circuit / fast overcurrent** (if supported)
- **Overtemperature (OT)**
- **Undertemperature (UT)** (important for charging)

Each item displays:
- Current reading (e.g., Vmin, Ipack, Tmax)
- Threshold value

---

## 3) Measurements: Hardware Requirements + Implementation

> Goal: reliably measure **pack current**, **pack/cell voltages**, and **temperature** with sufficient accuracy and update rate for UAV use.

### 3.1 Pack Current Measurement (Ipack)

**Hardware requirements**
- **Shunt resistor** (low-ohm, high-power, Kelvin sense):
  - Typical range: `0.5 mΩ – 2 mΩ` (choose based on max current and allowed power loss)
  - Power rating sized for worst-case: `P = I^2 R`
- **Current-sense amplifier**: e.g., **INA240** (good PWM rejection; robust for motor noise)
- **MCU ADC channel** (or external ADC) to digitize amplifier output

**Implementation**
- Place the **shunt** in series with pack negative (low-side) or positive (high-side) depending on design.
- Kelvin-route the sense traces to INA240 inputs.
- INA240 output → MCU ADC.
- Convert ADC to current:
  - `Vout = Gain × Vshunt + Vref_offset`
  - `Vshunt = I × Rshunt`
  - => `I = (Vout − offset) / (Gain × Rshunt)`
- Filtering:
  - Use analog RC at amplifier output + digital low-pass filter
  - Use a faster path (less filtering) for overcurrent detection if needed

**Viewer fields derived**
- Ipack, Ppack, coulombs integrated (Ah), rolling average current

---

### 3.2 Pack Voltage Measurement (Vpack)

**Hardware requirements**
- **Resistor divider** to scale pack voltage into ADC range
- MCU ADC channel (or external ADC)

**Implementation**
- Divider ratio: choose so max pack voltage maps below ADC reference with margin
- Add RC filter to reduce switching noise
- Convert:
  - `Vpack = Vadc × (Rtop + Rbottom) / Rbottom`
- Calibrate divider using measured reference voltages (offset/gain correction)

**Viewer fields derived**
- Vpack, power, and sanity checks vs sum(cell voltages)

---

### 3.3 Individual Cell Voltage Measurement (V1..Vn)

**Hardware requirements**

**Option A — Dedicated cell monitor IC (recommended)**
- A multi-cell battery monitor IC (typical 6–18 cells; depends on pack)
- Often uses isolated daisy-chain comms (isoSPI/SPI) in larger systems
- Pros: best accuracy, built-in balancing support, robust safety
- Cons: more integration effort

**Viewer fields derived**
- V1..Vn, Vmax, Vmin, ΔV imbalance, UV/OV per-cell flags

---

### 3.4 Temperature Measurement (T1..Tk)

**Hardware requirements**
- **NTC thermistors** (common: 10k NTC) placed at:
  - near hottest cell(s)
  - near MOSFETs / power path
  - pack surface (optional)
- **Resistor divider** into ADC, or dedicated temperature ICs (optional)

**Implementation**
- Thermistor + pull-up resistor → ADC
- Convert ADC to resistance → temperature using:
  - Beta equation or lookup table
- Filter readings and detect sensor faults:
  - open-circuit / short-circuit bounds

**Viewer fields derived**
- T1..Tk, Tmax, OT/UT flags, sensor-fault indicators

---

## 4) State Estimation Outputs (Software → Viewer)

### 4.1 Coulomb Counting SOC
- Integrate current over time:
  - `Ah_used += Ipack * dt / 3600`
- SOC:
  - `SOC_cc = 1 − (Ah_used / Ah_nominal)`
- Reset/trim using:
  - known full-charge detection
  - occasional OCV-based correction when resting (low current)

### 4.2 Kalman Filter SOC (ECM model)
- Use an equivalent circuit model (ECM):
  - OCV(SOC) mapping
  - series resistance R0
  - optional RC pair(s) for transient behavior
- Inputs: Ipack, Vpack (or sum of cell voltages), temperature (optional)
- Output: SOC_kf (+ confidence metric)

**Viewer fields**
- SOC_kf, SOC_cc, disagreement metric, confidence/health indicator (optional)

---

## 5) Update Rates
- Current & voltage: **10–50 Hz** (viewer can render at 10–20 Hz)
- Cell voltages: **2–10 Hz** (slower is usually fine)
- Temperature: **1–5 Hz**
- Fault flags: **immediate / event-driven** + periodic refresh

---

## 6) Data Link (BMS → Dashboard)
- UART (simple), CAN (robust), or USB-serial
- Packet includes:
  - timestamp
  - Vpack, Ipack, Ppack
  - V1..Vn
  - T1..Tk
  - SOC_cc, SOC_kf
  - fault bitmask + thresholds

---

## 7) Acceptance Criteria (What “Done” Looks Like)
- Dashboard updates continuously with no freezes
- Clearly indicates **fault/warning/normal**
- Shows cell imbalance and minimum cell voltage prominently
- SOC displayed from both estimators with consistent sign conventions
- Logs faults with timestamps for debugging
