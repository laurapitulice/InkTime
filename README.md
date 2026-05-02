# ⌚ "InkTime smartwatch" - PCB layout in Fusion 360
InkTime is an e-paper smartwatch designed for month-class battery life, sunlight-readable time-first UX, and essential phone notifications with basic activity
tracking.
Developed in Autodesk Fusion 360, the PCB layout adheres to the precise dimensions and mounting requirements of the wearable chassis, ensuring a seamless fit and optimal component placement.

---

## Hardware Diagram
![Hardware Diagram](Images/block_diagram.png)

This is an open-hardware wearable based on the nRF52840 microcontroller.

---

## System Architecture

### 1. Microcontroller (MCU)
- **Chip:** Nordic nRF52840 (Cortex-M4F @ 64MHz)
- **Memory:** 1MB Flash, 256KB RAM
- **Radio:** Bluetooth 5.4 Low Energy with integrated ceramic antenna
- **Oscillators:** 32MHz (operational) and 32.768kHz (sleep mode/RTC)

### 2. Power Management
- **Li-Po Charger:** BQ25180 (I2C control, programmable current)
- **Voltage Regulator:** RT6100 Buck-Boost (stable 3.3V output)
- **Battery Monitoring:** MAX17048 Fuel Gauge (ModelGauge algorithm)
- **Charging Interface:** USB-C with USBLC6-2 ESD protection

### 3. Display (E-Paper)
- **Type:** Electronic Paper Display (EPD)
- **Interface:** SPI + Power Gating Control (PWR_EPD)
- **Drive Circuit:** On-board integrated Boost converter (MBR0530 Diodes, L5 10uH) for VGH/VGL generation

### 4. Sensors and Feedback
- **IMU:** BMA421 (Ultra-low power accelerometer connected via I2C)
- **Haptic:** DRV2605 (Vibration driver with integrated effects library via I2C)
- **Input:** 3 mechanical buttons (UP, ENTER, DOWN) with filtering circuits

### 5. Communication Interfaces (I2C Mapping)
| Component | Function | Interface |
| :--- | :--- | :--- |
| **BMA421** | Motion Sensor | I2C |
| **MAX17048** | Battery Status | I2C |
| **DRV2605** | Vibration Motor | I2C |
| **BQ25180** | Charging | I2C |

---

## nRF52840 Pin Mapping

| Pin nRF52840 | Signal      | Component | Interface |
|-------------|------------|------------|----------|
| P0.00/XL1   | XL1        | Crystal X2 (32.768kHz) | XTAL |
| P0.01/XL2   | XL2        | Crystal X2 (32.768kHz) | XTAL |
| P0.05/AIN3  | EPD_CS     | E-Paper (J1 FPC) | SPI CS |
| P0.06       | SDA        | BMA423, BQ25180, MAX17048, DRV2605 | I2C SDA |
| P0.07       | SCL        | BMA423, BQ25180, MAX17048, DRV2605 | I2C SCL |
| P0.08       | IMU_INT1   | BMA423 | GPIO Input |
| P1.08       | IMU_INT2   | BMA423 | GPIO Input |
| P0.11       | PMIC_INT   | BQ25180 | GPIO Input |
| P0.12       | HAPTIC_EN  | DRV2605 | GPIO |
| VBUS        | VBUS       | USB-C (J4) | Power |
| D-          | D-         | USB-C (J4) / USBLC6 | USB |
| D+          | D+         | USB-C (J4) / USBLC6 | USB |
| P0.13       | SW_UP      | Buton Up | GPIO Input |
| P0.14       | SW_ENT     | Buton Enter | GPIO Input |
| P0.15       | EPD_DC     | E-Paper (J1 FPC) | SPI DC |
| P0.16       | EPD_RST    | E-Paper (J1 FPC) | GPIO |
| P0.17       | EPD_BUSY   | E-Paper (J1 FPC) | GPIO Input |
| P0.18/RESET | RESET      | TC2030-IDC | SWD/GPIO |
| SWDCLK      | SWDCLK     | TC2030-IDC | SWD |
| SWDIO       | SWDIO      | TC2030-IDC | SWD |
| P1.02       | SW_DN      | Buton Down | GPIO Input |
| P0.10/NFC2  | ALERT      | MAX17048 | GPIO Input |
| ANT         | RF         | Antena 2450AT18B100E | RF |
| P0.02/AIN0  | SCK        | E-Paper (J1 FPC) | SPI SCK |
| P0.03/AIN1  | MOSI       | E-Paper (J1 FPC) | SPI MOSI |

---

## Hardware Design & PCB Layout

### 1. Component Placement Strategy
The layout process followed a "mechanical-first" approach to ensure perfect alignment with the enclosure:
* **Mechanical Constraints:** Critical components (USB-C connector, side buttons, and the E-Paper FPC connector) were placed first and **locked** to prevent accidental displacement during the routing process.
* **Logical Grouping:** Components were clustered according to their schematic blocks. ICs were placed centrally to their respective functional areas to minimize signal path lengths.
* **Top-Side Only:** All components are placed exclusively on the **TOP layer** to simplify manufacturing and thermal management.
* **Microcontroller:** The **nRF52840** is positioned relatively in the center of the board to maintain balanced access to all peripheral blocks it commands.
* **User Interface:** Test points and tactile buttons were strategically placed along the board edges for easy access during debugging and final assembly.

### 2. Design Rules & Routing
The board utilizes a **2-layer stack-up** with a focus on signal integrity and power stability:
* **Power & Ground:** 
    * Power traces are maintained at a width of **≥ 0.3 mm**, while signal traces are **≥ 0.15 mm**.
    * **Daisy-chaining** was avoided for power; instead, a **branching topology** was used to prevent ground/power loops.
    * Solid **GND polygon pours** are implemented on both Top and Bottom layers, connected by stitching vias to ensure a low-impedance return path.
* **Decoupling:** 100nF capacitors are placed as close as possible to the power pins of each integrated circuit to filter high-frequency noise.
* **Geometry:** Strictly utilized **45° angles** and smooth miters; 90° bends were eliminated to prevent signal reflections and manufacturing "acid traps."
* **Trace Optimization:** Routes were kept as short as possible. Via usage was minimized, only added when top-layer density required a transition to the bottom layer. Traces were kept away from sensitive areas like the crystal oscillator and inductors.

### 3. RF & EMI Management
Special precautions were taken for the wireless and power sections:
* **Antenna Integrity:** The ceramic antenna is isolated in the top-right corner. A strict **keep-out zone** (no copper, traces, or vias) was maintained around the antenna to prevent signal degradation and interference.
* **Inductor Orientation:** Series inductors (**L2, L3**) are positioned **perpendicularly** (90°) to each other to minimize mutual magnetic coupling and unwanted electromagnetic field amplification.
* **Clean Zones:** No traces were routed underneath inductors, the crystal oscillator, or mounting holes to maintain signal purity.

### 4. Design Rule Check (DRC) & Intentional Exceptions
The design was verified against custom DRC rules. Some errors were manually reviewed and accepted based on specific design requirements:
* **Dimension/Clearance Errors:** The USB-C connector and buttons extend slightly beyond the PCB edge for enclosure alignment. These "Dimension" errors are intentional and neglected.
* **Via-in-Pad:** In extremely dense areas where traditional routing was inaccessible, **0.15 mm micro-vias** were placed within pads to facilitate connectivity.
* **3D Modeling:** 3D models were applied to bulky and critical components for enclosure fit-checks; small passives use standard footprint placeholders.

### 5. Final Verification
* **Clear Silkscreen:** All labels and component designators were cleaned and oriented for high legibility (clear and readable).
* **Connectivity Audit:** A final manual and automated netlist audit was performed to ensure 100% routing completion.
* **Manufacturing Ready:** DRC was cleared of all non-intentional errors, ensuring the board meets professional fabrication capabilities.
  
---

## Debug and Programming
- **SWD:** Programming and debugging via Tag-Connect TC2030 connector
- **USB:** Serial communication and charging

---

## Power Consumption and Battery Life

The device is optimized for ultra-low power operation, leveraging the nRF52840's sleep states and power-gating techniques for high-voltage peripherals.

### Estimated Current Consumption
| Operating Mode | Active Components | Estimated Current |
| :--- | :--- | :--- |
| **Deep Sleep** | MCU (System OFF), Fuel Gauge (Sleep) | ~10 µA |
| **Standby (Idle)** | MCU (System ON), IMU (Low-power) | ~0.8 mA |
| **EPD Refresh** | MCU, EPD Boost Circuit, Display | ~12 mA (peak) |
| **Haptic Alert** | MCU, Haptic Driver + Motor | ~80 mA (peak) |

### Battery Life Estimation
Based on a **200mAh** Li-Po battery:
- **Static Time (Always-on):** Over 1 year (EPD retains the image without power).
- **Typical Usage:** ~10-12 days (with BLE active and 1 refresh/hour).
- **Heavy Usage:** ~4-5 days (frequent haptic notifications and updates).
  
---

## Bill of Materials (BOM)

| Component | Description | Qty | Manufacturer | Part Number | Datasheet |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Microcontroller** | nRF52840 (AQFN-74) | 1 | Nordic Semiconductor | NRF52840_QF | [Link](https://files.seeedstudio.com/wiki/XIAO-BLE/Nano_BLE_MCU-nRF52840_PS_v1.1.pdf) |
| **Antenna** | 2.45GHz Chip Antenna | 1 | Johanson Technology | 2450AT18B100E | [Link](https://www.johansontechnology.com/datasheets/2450AT18B100.pdf) |
| **Accelerometer** | Triaxial low-g 12bit Sensor | 1 | Bosch Sensortec | BMA423 | [Link](https://watchy.sqfmi.com/assets/files/BST-BMA423-DS000-1509600-950150f51058597a6234dd3eaafbb1f0.pdf) |
| **Battery Charger** | Li-Ion/Polymer Charger IC | 1 | Texas Instruments | BQ25180YBGR | [Link](https://www.ti.com/product/BQ25180) |
| **Haptic Driver** | Driver for ERM/LRA | 1 | Texas Instruments | DRV2605YZFR | [Link](https://www.ti.com/product/DRV2605) |
| **Voltage Regulator** | Buck-Boost DC-DC | 1 | Richtek | RT6160AWSC | [Link](https://www.richtek.com/Products/Switching%20Regulators/Buck-Boost%20Converter/RT6160A) |
| **Fuel Gauge** | 1-Cell/2-Cell Fuel Gauge | 1 | Analog Devices | MAX17048G+T10 | [Link](https://www.analog.com/en/products/max17048.html) |
| **Connector (FPC)** | 0.5mm FPC RA SMT 24Ckt | 1 | Molex | 503480-2400 | [Link](https://www.molex.com/en-us/products/part-detail/5034802400) |
| **Connector (USB)** | USB Type-C 16-pin | 1 | Kinghelm | KH-TYPE-C-16P | [Link](https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8588905154556923904) |
| **ESD Protection** | Low Cap. ESD Protection | 1 | STMicroelectronics | USBLC6-2SC6Y | [Link](https://www.st.com/en/protection-devices/usblc6-2.html) |
| **MOSFET (P)** | P-channel 20V/4.2A | 1 | Diodes Inc. | DMG2305UX-7 | [Link](https://www.lcsc.com/datasheet/C469327.pdf) |
| **MOSFET (N)** | N-channel 30V 1.5A | 1 | Vishay Siliconix | SI1308EDL-T1-GE3 | [Link](https://www.lcsc.com/datasheet/C469327.pdf) |
| **Schottky Diode** | 30V 0.5A Schottky | 3 | ON Semiconductor | MBR0530 | [Link](https://www.onsemi.com/pdf/datasheet/mbr0530t1-d.pdf) |
| **Inductor** | SMD Inductor 0.47µH | 1 | TDK | FTC252012SR47MBCA | [Link](https://jlcpcb.com/partdetail/6763488-FTC252012SR47MBCA/C5832368) |
| **Inductor** | Power Inductor 68uH | 1 | Wurth Electronics | 744043680 | [Link](https://www.we-online.com/en/components/products/WE-TPC) |
| **Switches** | Tactile Switches | 3 | Panasonic | EVP-AKE31A | [Link](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2301111010_PANASONIC-EVPAKE31A_C569760.pdf) |
| **Crystals** | 32MHz & 32.768kHz | 2 | Generic/Nordic | X1, X2 | [Link1](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2312080231_NDK-NX2016SA-32MHZ-EXS00A-CS11336_C6134317.pdf), [Link2](https://jlcpcb.com/partdetail/SeikoEpson-FC_135_32_7680KAA3/C2650472) |
| **Test Connector** | CABLE ADAPTER 6 POS | 1 | Tag Connect | TC2030-IDC | [Link](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/2019/12/TC2030-IDC-Datasheet-Rev-B.pdf) |
