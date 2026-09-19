# Precision Wireless Data Acquisition (DAQ) Card using ESP32-S3

[![Altium Designer](https://img.shields.io/badge/EDA-Altium%20Designer-brightgreen?style=for-the-badge&logo=altiumdesigner)](https://www.altium.com/)
[![Resolution](https://img.shields.io/badge/ADC%20Resolution-16--Bit%20%CE%94%CE%A3-blue?style=for-the-badge)](#analog-signal-chain-architecture)
[![Instrumentation](https://img.shields.io/badge/Front--End-TI%20INA333%20(100dB%20CMRR)-orange?style=for-the-badge)](#1-precision-instrumentation-amplifier-ti-ina333)
[![Wireless](https://img.shields.io/badge/Telemetry-Wi--Fi%204%20%2F%20BLE%205.0-red?style=for-the-badge)](#microcontroller--wireless-telemetry-esp32-s3)
[![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)](LICENSE)

A high-accuracy, mixed-signal **Wireless Data Acquisition (DAQ) Platform** engineered for industrial sensor instrumentation, laboratory telemetry, and precision measurements. 

The card pairs a high-performance **Espressif ESP32-S3** microcontroller (dual-core Xtensa® 32-bit LX7 @ 240 MHz) with an analog signal chain comprising a **16-bit Delta-Sigma ADC**, **precision micro-power instrumentation amplifier**, **12-bit DAC**, and **dedicated shunt voltage reference**.

---

## Analog Signal Chain Architecture

```
                                  ANALOG FRONT-END (AFE)
                      ┌──────────────────────────────────────────────┐
                      │                                              │
                      │  Differential   ┌─────────────────────────┐  │
 Differential Sensor ─┼─ Sensor Inputs ─>│ Texas Instruments INA333│  │
 (Bridge, RTD, Shunt) │  (V+ / V-)      │ Precision InAmp (100dB) │  │
                      │                 └───────────┬─────────────┘  │
                      │                             │ Conditioned    │
                      │                             │ Analog Signal  │
                      │                             ▼                │
 Single-Ended /       │                 ┌─────────────────────────┐  │
 Multichannel Sensors ┼─ Analog Channels─>│ Texas Instruments ADS1115│  │
 (AIN0 - AIN3)        │  (0 - 3.3V / 5V)│ 16-Bit Delta-Sigma ADC  │  │
                      │                 │ (PGA: +/-256mV to 6.144V│  │
                      │                 └───────────┬─────────────┘  │
                      │                             │                │
                      │                 ┌───────────┴─────────────┐  │      DIGITAL / WIRELESS
                      │                 │ Microchip LM4040        │  │ ┌────────────────────────┐
                      │                 │ 2.5V Precision VREF     │  │ │ ESP32-S3 Dual-Core SoC │
                      │                 └─────────────────────────┘  │ │                        │
                      │                                              │ │ - 240 MHz Xtensa LX7   │
                      │  Closed-Loop    ┌─────────────────────────┐  │ │ - 8MB Flash            │
 Excitation / Offset  │  Analog Output  │ Microchip MCP4725       │  │ │ - 2.4GHz Wi-Fi (HTTP/  │
 Control Signal      <┼─────────────────┤ 12-Bit DAC with EEPROM  │<─┼─┤   MQTT / WebSockets)   │
                      │                 │ (I2C Interface)         │  │ │ - Bluetooth 5.0 (BLE)  │
                      │                 └───────────▲─────────────┘  │ │                        │
                      │                             │ I2C Bus        │ │                        │
                      └─────────────────────────────┼────────────────┘ └───────────▲────────────┘
                                                    │ (SDA / SCL)                  │
                                                    └──────────────────────────────┘
```

---

## Key Hardware Subsystems

### 1. Precision Instrumentation Amplifier: Texas Instruments INA333
* **Low Noise & Low Drift:** Features maximum offset voltage of **25 µV**, near-zero drift of **0.5 µV/°C**, and a quiescent current of just **50 µA**.
* **High Common-Mode Rejection:** 100 dB CMRR (G ≥ 10), rejecting common-mode electrical noise in industrial environments.
* **Gain Programmability:** Single external gain resistor ($R_G$) sets closed-loop gain from 1 to 1000:
  $$G = 1 + \frac{100\text{ k}\Omega}{R_G}$$
* **Application:** Ideal for amplifying microvolt signals from Wheatstone bridge load cells, thermocouples, strain gauges, and precision current-sensing shunts.

### 2. 16-Bit Delta-Sigma ADC: Texas Instruments ADS1115
* **High-Resolution Digitization:** 16-bit resolution ($2^{16} = 65,536$ discrete codes) with data rates programmable from 8 SPS to 860 SPS.
* **Input Multiplexer:** Flexible analog multiplexer supporting either **4 single-ended inputs** (`AIN0` - `AIN3`) or **2 fully differential pairs**.
* **On-Chip Programmable Gain Amplifier (PGA):** Accommodates full-scale input ranges from **$\pm 256\text{ mV}$** up to **$\pm 6.144\text{ V}$**, allowing direct digitization of both millivolt-level sensor signals and industrial 0-5V signals.
* **Internal Oscillator & Reference:** Integrated low-drift reference ($8\text{ ppm}/^\circ\text{C}$) and digital comparator with alert interrupt output (`ALERT/RDY`).

### 3. Precision Voltage Reference: Microchip LM4040
* **Micropower 2.5V Shunt Reference:** Low-tolerance (1%) precision reference (`U6`) provides an ultra-stable benchmark voltage for ratiometric sensor excitation and ADC calibration.
* Sub-microamp knee current capability ensuring tight voltage stability regardless of battery state.

### 4. 12-Bit Analog Output DAC: Microchip MCP4725
* **Analog Waveform & Excitation Generation:** 12-bit buffered voltage-output DAC (`U4`) controlled over the shared I2C bus.
* **Non-Volatile EEPROM:** Retains DAC register settings across power cycles, enabling autonomous baseline offset injection, bridge sensor excitation, or analog current loop setpoints.

### 5. Overvoltage & Transient Surge Protection
* **Littelfuse SMAJ26CA:** High-reliability **400W bidirectional TVS diode** (`D1`) guarding the primary power rail against reverse voltage, electrostatic discharge (ESD), and high-energy inductive load transients.
* High-frequency decoupling network employing AEC-Q200 qualified **Taiyo Yuden 100nF X7R ceramic capacitors** (`C1` - `C4`) positioned right at the power pins of each analog IC.

### 6. Power Supply Regulation
* **AMS1117-3.3 Linear Regulator:** 1A low-dropout linear regulator providing clean 3.3V power to the analog front-end and the ESP32-S3 module.
* High-efficiency green indicator LED (**Avago HSMG-C170**) provides immediate visual confirmation of rail power.

---

## Bill of Materials (BOM)

A complete, machine-readable BOM is provided in [`Documentation/Bill_of_Materials.csv`](Documentation/Bill_of_Materials.csv).

| Designator | Quantity | Component Name | Description | Manufacturer | MPN | Package |
|:---|:---:|:---|:---|:---|:---|:---|
| **U1** | 1 | ESP32-S3 Mini DevBoard | Dual-Core Xtensa LX7 @ 240MHz, 8MB Flash, Wi-Fi 4/BLE 5.0 | Espressif Systems | `ESP32-S3-DEVKITM-1-N8` | Module / Header |
| **U2** | 1 | TI ADS1115 | 16-Bit Delta-Sigma ADC, Integrated PGA & Reference, I2C | Texas Instruments | `ADS1115IRUGR` | QFN-10 (1.5x2mm) |
| **U3** | 1 | TI INA333 | Micro-Power Precision Instrumentation Amplifier, 25µV Offset | Texas Instruments | `INA333AIDGKT` | 8-VSSOP |
| **U4** | 1 | Microchip MCP4725 | 12-Bit DAC with I2C and On-Board Non-Volatile EEPROM | Microchip Technology | `MCP4725A0T-E/CH` | SOT-23-6 |
| **U5** | 1 | AMS1117-3.3 | 1A Low-Dropout Linear Voltage Regulator, 3.3V Output | Advanced Monolithic Systems | `AMS1117-3.3` | SOT-223 |
| **U6** | 1 | Microchip LM4040 | Precision 2.5V 1% Micropower Shunt Voltage Reference | Microchip Technology | `LM4040DYM3-2.5-TR` | SOT-23-3 |
| **D1** | 1 | Littelfuse SMAJ26CA | 400W 26V Bidirectional Surface Mount TVS Surge Diode | Littelfuse Inc. | `SMAJ26CA` | DO-214AC (SMA) |
| **DS1** | 1 | Green LED Indicator | High-Efficiency Green SMD LED, 2.2V 20mA (Power Status) | Avago / Broadcom | `HSMG-C170` | 0805 SMD |
| **J1** | 1 | 10-Pin Terminal Header | 10-Pin Vertical Male Breakout Header, 2.54mm Pitch | Würth Elektronik | `61301011121` | HDR 2x5 2.54mm |
| **C1-C4** | 4 | 100nF Decoupling Caps | Ceramic Capacitor 0.1µF 25V 10% X7R AEC-Q200 | Taiyo Yuden | `TMK107B7104KAHT` | 0603 SMD |
| **R4, R5** | 2 | 2.2kΩ Pullup Resistors | Resistor SMD 2.2kΩ 5% 1/16W (I2C Bus Pullups) | Yageo | `RC0402JR-072K2L` | 0402 SMD |
| **R3** | 1 | 100Ω Resistor | Resistor SMD 100Ω 1% 1/16W | Yageo | `RC0402JR-07100RL` | 0402 SMD |
| **R6** | 1 | 4.7kΩ Power Resistor | Resistor SMD 4.7kΩ 5% 0.5W Power Resistor | Yageo | `RC1210JR-074K7L` | 1210 SMD |

---

## Pinout & Connector Interface (`J1`)

The 10-pin interface header (`J1`) exposes all core analog and power connections:

| Pin # | Net Name | Signal Direction | Description |
|:---:|:---|:---:|:---|
| **1** | `AIN0 / VIN+` | Input | ADS1115 Differential Channel 0 (+) or INA333 Positive Input |
| **2** | `AIN1 / VIN-` | Input | ADS1115 Differential Channel 0 (-) or INA333 Negative Input |
| **3** | `AIN2` | Input | ADS1115 Single-Ended / Differential Channel 1 Input |
| **4** | `AIN3` | Input | ADS1115 Single-Ended / Differential Channel 1 Input |
| **5** | `DAC_OUT` | Output | Microchip MCP4725 12-Bit Analog Output Voltage |
| **6** | `VREF_2V5` | Output | LM4040 Precision 2.5V Reference Output |
| **7** | `SDA` | Bidirectional | I2C Serial Data line (shared bus with 2.2kΩ pullup) |
| **8** | `SCL` | Input (Clock) | I2C Serial Clock line (shared bus with 2.2kΩ pullup) |
| **9** | `+3.3V` | Power Output | Clean Regulated 3.3V System Rail |
| **10** | `GND` | Ground | System Analog & Digital Ground Return |

---

## Repository Structure

```
├── Hardware/
│   ├── VI_DAQCard.PrjPcb              # Altium project master
│   ├── vidaq.SchDoc                   # Complete analog & digital schematic
│   ├── vidaq.PcbDoc                   # Altium PCB layout document
│   ├── VI_DAQCard.BomDoc              # Altium LiveBOM document
│   └── VI_DAQCard.PrjPcbStructure     # Altium project structure file
├── Documentation/
│   └── Bill_of_Materials.csv          # Complete engineering BOM
└── README.md
```

---

## Authors & Credits

* **Aakashdip Dey** - Hardware Architecture, Schematic Capture, Analog Signal Chain Design
* **Rishabh** - Co-Designer & Collaborator

**Contact:**
* Location: Vellore, India
* GitHub: [@AakashdipDey](https://github.com/AakashdipDey)
