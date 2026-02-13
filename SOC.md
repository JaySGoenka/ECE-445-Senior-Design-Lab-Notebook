# Smart BMS – SOC Estimation Notes

## 1. System Overview

The goal of the Battery Management System (BMS) is to accurately estimate the State of Charge (SOC) of a Li-ion battery pack during both charging and deployment in a UAV system.

SOC estimation requires:
- Accurate current measurement
- Accurate voltage measurement
- Temperature monitoring
- A robust estimation algorithm (Coulomb counting + Kalman filter)

---

## 2. Current Measurement

### 2.1 Shunt Resistor

A shunt resistor is a precision, low-resistance component placed in series with the battery to measure current.

It works using Ohm’s Law:

V_shunt = I × R_shunt

Where:
- V_shunt is a small voltage drop (typically millivolts)
- I is battery current
- R_shunt is typically 1–5 mΩ

Important design trade-offs:
- Lower resistance → less power loss but smaller measurable voltage
- Higher resistance → better signal resolution but more heat dissipation

Power dissipation:
P = I²R

---

### 2.2 High-Side Current Sensing

The shunt resistor is placed on the high side (between battery positive and load).

Advantages:
- Detects short-to-ground faults
- Preserves system ground reference
- More suitable for UAV systems

---

### 2.3 Current Sense Amplifier (INA240)

The shunt voltage is amplified using a high-side current sense amplifier.

Requirements:
- High common-mode rejection
- PWM noise rejection
- Accurate gain
- Low offset voltage

---

### 2.4 Signal Conditioning

An RC low-pass filter is added before the ADC input to remove high-frequency PWM noise.

Cutoff frequency must balance:
- Noise suppression
- Transient response accuracy

---

### 2.5 ADC Requirements

ADC must provide:
- Sufficient resolution (12–16 bit)
- Stable reference voltage
- Low noise

Resolution impacts SOC accuracy directly.

---

## 3. Voltage Measurements

Voltage measurement is required for:

- Open Circuit Voltage (OCV) based SOC correction  
- Over-voltage protection  
- Under-voltage protection  
- Cell balancing decisions  
- Health monitoring  

In a UAV BMS, both **pack-level voltage** and **individual cell voltages** must be monitored.

---

### 3.1 Measurement Objectives

The voltage measurement subsystem must:

- Measure maximum pack voltage safely (e.g., 4S Li-ion ≈ 16.8 V max)
- Provide sufficient resolution for SOC correction
- Introduce minimal loading on the battery
- Maintain low noise during high PWM motor activity

---

### 3.2 Resistor Divider for Pack Voltage

Since the MCU ADC cannot tolerate voltages above its reference (typically 3.3 V), a resistor divider is used to scale down battery voltage.

Voltage divider equation:

V_ADC = V_BAT × (R2 / (R1 + R2))

Example design for 4S Li-ion:

- Max battery voltage: 16.8 V  
- ADC reference: 3.3 V  
- Target ADC max: 3.0 V (safety margin)

Design ratio:

R2 / (R1 + R2) ≈ 0.18

Example component selection:

- R1 = 82kΩ  
- R2 = 18kΩ  

---

### 3.3 Design Trade-offs

#### 1. Power Dissipation

The divider continuously draws current:

I = V_BAT / (R1 + R2)

For 100kΩ total at 16V:

I ≈ 160 µA

This is acceptable for UAV systems but should be minimized in ultra-low-power designs.

---

#### 2. Accuracy

Voltage measurement error directly impacts SOC correction.

Error sources:
- Resistor tolerance (1%, 0.1%)
- Temperature coefficient
- ADC reference drift
- PCB leakage
- Noise from PWM switching

Design recommendations:
- Use 0.1% resistors for improved SOC accuracy
- Use proper PCB layout with short traces
- Add RC filtering at ADC input

---

### 3.4 RC Filtering

A capacitor is added across R2 to create a low-pass filter.

Purpose:
- Remove high-frequency switching noise
- Stabilize ADC sampling

Cutoff frequency:

f_c = 1 / (2πR_eqC)

Must be chosen carefully:
- Too low → slow response to voltage transients
- Too high → insufficient noise filtering

Typical cutoff range: 10–100 Hz for SOC measurement.

---

### 3.5 Cell-Level Voltage Monitoring

For multi-cell packs, each cell must be monitored individually.

Reasons:
- Prevent overcharge (>4.2 V)
- Prevent undervoltage (<3.0 V)
- Enable passive or active balancing

For higher cell counts, dedicated battery monitor ICs are preferred over simple resistor dividers.

---

## 4. Temperature Sensing, ADC, and MCU

Temperature is critical in Li-ion systems because:

- Internal resistance increases with temperature
- Capacity varies with temperature
- High temperature accelerates degradation
- Thermal runaway risk exists

---

### 4.1 Temperature Sensing

Most BMS designs use:

- NTC thermistors (most common)
- Digital temperature sensors (less common for battery packs)

---

#### NTC Thermistor

An NTC (Negative Temperature Coefficient) thermistor decreases resistance as temperature increases.

It is typically placed:

- On the cell surface
- Near high-current MOSFETs
- Near the shunt resistor

---

#### Thermistor Measurement Circuit

The thermistor is used in a resistor divider configuration:

Vref ── R_fixed ──┬── ADC
                  │
               NTC thermistor
                  │
                 GND

The measured voltage corresponds to temperature via the Steinhart–Hart equation.

---

#### Why Temperature Matters for SOC

SOC algorithms require temperature compensation because:

- OCV vs SOC curves shift with temperature
- Internal resistance changes
- Capacity (C_nominal) varies

Without temperature compensation, SOC error increases significantly.

---

### 4.2 ADC Requirements

The Analog-to-Digital Converter (ADC) converts analog voltage signals into digital values.

The ADC must measure:

- Shunt amplifier output (current)
- Divider output (voltage)
- Thermistor output (temperature)

---

#### Resolution

Resolution determines the smallest measurable voltage change.

For a 12-bit ADC:

Resolution = V_ref / 4096

If V_ref = 3.3 V:

LSB ≈ 0.8 mV

Higher resolution (e.g., 16-bit external ADC) improves:

- Current precision
- SOC drift performance
- Kalman filter stability

---

#### Sampling Rate

Sampling must be high enough to:

- Capture current dynamics
- Support accurate Coulomb counting

Typical BMS sampling:
- 1–10 kHz for current
- 10–100 Hz for voltage and temperature

---

#### ADC Error Sources

- Quantization error
- Reference drift
- Offset error
- Gain error
- Noise

These errors propagate directly into SOC estimation.

---

### 4.3 MCU (Microcontroller Unit)

The MCU is the central controller of the BMS.

It performs:

- ADC data acquisition
- Coulomb counting
- Kalman filtering
- Protection logic
- Communication with flight controller

Common MCU vendors include:

- STMicroelectronics  
- NXP Semiconductors  
- Microchip Technology  

---

#### MCU Requirements for This BMS

- Multiple ADC channels
- DMA support for continuous sampling
- Floating-point unit (for Kalman filter efficiency)
- SPI/I2C communication interfaces
- Low power modes
- Sufficient RAM for filtering and state estimation

---

### 4.4 Firmware Responsibilities

The firmware must:

1. Sample current at fixed interval Δt  
2. Integrate current for Coulomb counting  
3. Periodically correct SOC using voltage and temperature models  
4. Apply safety protections  
5. Log and transmit diagnostic data  

---

## System Measurement Chain Summary

Battery  
→ Sensors (Shunt / Divider / Thermistor)  
→ Analog Conditioning  
→ ADC  
→ MCU  
→ SOC Estimation Algorithm  

The ultimate accuracy of SOC estimation is fundamentally limited by the precision, stability, and noise performance of these measurement subsystems.



## 5. SOC Estimation Methods

### 5.1 Coulomb Counting

SOC(t) = SOC(t0) - (1/C_nominal) ∫ I(t) dt

Limitations:
- Accumulates drift over time
- Sensitive to current measurement error

---

### 5.2 Kalman Filter

Used to correct drift by:
- Comparing measured voltage
- Using battery model (OCV + internal resistance)
- Updating SOC estimate

---

## 6. Error Sources

- Shunt tolerance
- Amplifier offset
- ADC quantization
- Temperature variation
- Battery aging

---

## 7. Design Requirements for UAV

- High dynamic current handling
- Minimal power loss
- High noise immunity
- Real-time SOC update
- Safe operation in both charging and deployment modes
