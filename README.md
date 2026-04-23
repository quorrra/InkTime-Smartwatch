
# InkTime — Open Source Smartwatch

InkTime is a low-cost, open-source smartwatch designed for extreme battery life and hackability.  
It features a **1.54" E-Paper display**, a **Nordic nRF52840 BLE SoC**, and a high-efficiency power system.

---

## 1) Block Diagram

The hardware architecture revolves around the nRF52840 MCU and a power system designed for month-long battery life.

````text
[ USB Type-C ]                              [ LiPo Battery (250mAh) ]
                 | 5V VBUS                                      |
                 v                                              |
          +---------------+                                     |
          |    BQ25180    | BAT                                 |
          | (Charger IC)  |<------------------------------------+
          +---------------+                                     v
                 | VSYS                                 +---------------+
                 v                                      |   MAX17048    |
          +---------------+                             | (Fuel Gauge)  |
          |    RT6160     |                             +---------------+
          | (Buck-Boost)  |                                     ^
          +---------------+                                     |
                 | 3.3V System Rail (VDD_3V3)                   |
                 |                                              |
 +---------------+----------------------------------------------+-----+
 |               |                                              |     |
 |               v                                              |     |
 |        +-------------+         I2C Bus                       |     |
 |        |  nRF52840   |<======================================+     |
 |        |    MCU      |                                             |
 |        |             |=== I2C ===> [ BMA423 Accelerometer ]        |
 |        |             |=== I2C ===> [ DRV2605L Driver ]-->[ Motor ] |
 |        |             |=== SPI ===> [ 1.54" E-paper Display ]       |
 |        |             |                       ^                      |
 |        |             |-- GPIO (Gate)         | VEPD (Switched Pwr) |
 |        +-------------+      |                |                      |
 |          ^  ^  ^  |         v                |                      |
 |          |  |  |  | +---------------+        |                      |
 +---------(|--|--|)-+-|  PFET Switch  |--------+                      |
            |  |  |  | +---------------+                               |
 [Up]-------+  |  |  |                                                 |
           [Down] |  +--- [ 2.4GHz Antenna ] - - BLE - - [ Companion Phone ]
                  |
              [Enter/Esc]
````

---

## 2) Bill of Materials (BOM)

All components are selected from the JLC Parts library for easy manufacturing and assembly.

| RefDes | Component      | Description               | JLC Part # | Datasheet |
|--------|----------------|---------------------------|------------|-----------|
| U1     | nRF52840       | BLE SoC, Cortex-M4F       | [C137156](https://jlcpcb.com/partdetail/C190794)    | BLE, Native USB      |
| IC1    | BQ25180        | Li-Ion Charger            | [C2907455](https://jlcpcb.com/partdetail/C3682423)   |Safe USB power-path     |
| IC9    | RT6160AWSC     | Buck-Boost Converter      | [C2834376](https://jlcpcb.com/partdetail/C7065276)   | 3.3V System Rail      |
| U3     | MAX17048G      | I2C Fuel Gauge            | [C146603](https://jlcpcb.com/partdetail/C2682616)    | 1-Cell Battery Monitor      |
| IC3    | BMA423         | 3-axis Accelerometer      | [C224345](https://jlcpcb.com/partdetail/C5242966)    | Embedded pedometer      |
| IC4    | DRV2605YZFR    | Haptic Driver             | [C146312](https://jlcpcb.com/partdetail/C527464)    | For ERM motor     |
| J4     | KH-TYPE-C-16P  | USB-C Connector           | [C319148](https://jlcpcb.com/partdetail/C7528806)    |High-side power switch    |

---

## 3) Hardware Functionality

### 3.1 Power Management System
- The watch is powered by a **250mAh LiPo battery**.
- **Charging:** BQ25180 handles USB charging (5V VBUS) and system power-path management.
- **Regulation:** RT6160 buck-boost ensures a stable **3.3V rail** even when battery voltage drops below 3.3V.
- **Fuel Gauge:** MAX17048 uses ModelGauge for accurate **SoC (%)** via I2C, without a sense resistor.

### 3.2 Processing & Connectivity
The **nRF52840** is the central processor, running BLE for smartphone notifications and controlling peripherals over I2C/SPI.

### 3.3 Display & Haptics
- **E-Paper Display:** 1.54", ultra-low power (0W image retention), controlled via SPI.
- **Haptics:** DRV2605 drives an LRA motor for tactile alerts and alarms.

---

## 4) nRF52840 Pin Mapping

| Pin   | Signal Name | Interface  | Function |
|-------|-------------|------------|----------|
| P0.13 | SW_UP       | GPIO (In)  | Up navigation button |
| P0.14 | SW_ENT      | GPIO (In)  | Enter/Select button |
| P1.02 | SW_DN       | GPIO (In)  | Down navigation button |
| P0.08 | I2C_SDA     | I2C Data   | Shared SDA (IMU, Charger, Gauge, Haptic) |
| P1.09 | I2C_SCL     | I2C Clock  | Shared SCL (IMU, Charger, Gauge, Haptic) |
| P0.20 | EPD_RESET   | GPIO (Out) | Display hardware reset |
| P0.22 | EPD_BUSY    | GPIO (In)  | Display status pin |
| P0.12 | HAPTIC_EN   | GPIO (Out) | Enable for DRV2605 haptic driver |
| P0.01 | EPD_DC      | GPIO (Out) | Data/Command selector for E-paper |
| P0.03 | EPD_CS      | SPI CS     | Chip select for E-paper display |

---

## 5) Design & Manufacturing Details

### 5.1 PCB Stackup & Constraints
- **Stackup:** 4-layer PCB (Top, GND Plane, Power Plane, Bottom)
- **Thickness:** 1.0 mm (wearable-friendly)
- **Antenna:** 2.4 GHz PCB antenna at board edge with ground clearance for BLE performance

### 5.2 Component Selection
- **Passives:**  
  - Resistors: 0201 SMD  
  - Capacitors ≤100nF: 0201  
  - Capacitors >100nF: 0402 (unless otherwise specified)
- **Placement:** All components on TOP layer

### 5.3 Routing Rules
- **Power traces** (VCC, VUSB, VBUS, 3V3): 0.3 mm
- **Signal traces:** minimum 0.15 mm
- **Grounding:** solid Layer 2 GND plane + via stitching for EMI/thermal performance

---

## 6) Design Log & Decisions

- **4-layer decision:** moved from 2 to 4 layers to route the 24-pin E-paper connector.
- **GND plane:** Layer 2 kept continuous for RF reference and improved signal integrity.
- **Mechanical fit:** verified in 3D enclosure model (USB-C and buttons aligned with housing).