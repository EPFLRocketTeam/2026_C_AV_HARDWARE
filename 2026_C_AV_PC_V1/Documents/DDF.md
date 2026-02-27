# Introduction

This document describes the design and function of the Propulsion Computer (PRC) PCB. It is intended to provide sufficient detail for a reader to understand the board's operation and reproduce it. The PRC is responsible for controlling the propulsion system, acquiring sensor data, and logging flight data to an SD card. It interfaces with the main avionics stack via a spine connector and supports standalone operation through a USB interface.


## Definitions and Abbreviations

- **CAN**: Controller Area Network.
- **DDF** : Design Definition File.
- **ESD** : Electrostatic Discharge
- **I2C**: Inter-Integrated Circuit
- **LDO**: Low-Dropout Regulator, a DC linear voltage regulator that creates the 3.3V power rail (VDD).
- **MCU**: Microcontroller Unit.
- **OTG_USB_FS / HS**: On-The-Go USB Full-Speed / High-Speed, the USB peripheral controllers within the MCU.
- **PCB**: Printed Circuit Board.
- **PRC**: Propulsion Computer, the avionics board responsible for controlling the propulsion system and logging flight data.
- **PT1000**: A type of Resistance Temperature Detector (RTD) with a resistance of 1000 Ω at 0°C, used for precise temperature measurements.
- **PWM**: Pulse Width Modulation, a technique for controlling power delivery or signal levels by varying the duty cycle of a digital signal.
- **RS232**: Recommended Standard 232. A point-to-point serial communication standard using single-ended voltage levels (typically ±3 V to ±15 V) for asynchronous data exchange between devices such as computers and peripherals.
- **SCL**: Serial Clock, line responsible for controlling the timing of data transfers between devices on the I2C bus.
- **SDA**: Serial Data, line used for data transmission between devices on the I2C bus.
- **SDIO**: Secure Digital Input/Output, the 4-bit bus protocol used for high-speed data transfer with the SD Card.
- **SDMMC**: Secure Digital MultiMedia Card, the MCU peripheral interface for the SD Card.
- **SPI** : Serial Peripheral Interface
- **SWD**: Serial Wire Debug, the standard two-pin interface used for programming and debugging the MCU.
- **UART / USART**: Universal Asynchronous/Synchronous Receiver/Transmitter, a common serial protocol.
- **USB**: Universal Serial Bus.
- **VCP**: Virtual COM Port, a method of communicating with the MCU over USB, appearing as a standard serial port.
- **VDD**: Drain Voltage, the primary 3.3V power supply rail for the MCU and peripherals.
- **TVS** : Transient Voltage Suppression

{.grid-list}

## Applicable and Reference Documents


# Requirements

## table {.tabset}
### Requirement Name
- [2026_C_SE_AV_REQ_??]()**Requirement Title**
Requirement description
{.links-list}

### Requirement Name
- [2026_C_SE_AV_REQ_??]()**Requirement Title**
Requirement description
{.links-list}

### Requirement Name
- [2026_C_SE_AV_REQ_??]()**Requirement Title**
Requirement description
{.links-list}

# Functional Description
The Propulsion Computer (PRC) is a critical component of the avionics system, responsible for controlling the propulsion system, acquiring sensor data, and logging flight data. It interfaces with the main avionics stack via a spine connector and supports standalone operation through a USB interface.

Here is the block diagram presenting an overview of the different functions of the FDR.

## Block Diagram
The PRC is built around an STM32H743VIT6 microcontroller, which executes the main embedded software responsible for:
- Acquiring data from various sensors (e.g., pressure, temperature, current) through I2C and analogue inputs.
- Controlling the propulsion system through PWM outputs and digital GPIOs.
- Communicating with the main avionics stack via a spine connector using UART and CAN protocols.
- Logging flight data to an SD card at a rate of 10 Hz.

The 10 Hz logging rate ensures sufficient temporal resolution for post-flight analysis while managing storage capacity and power consumption effectively.

![PRC_BDI.png](/competition/Firehorn_2/Avionics/prc/prc_bdi.png){.align-center}

## Power management
The PCB is designed to operate from two independent 5 V input sources:
- 5V Spine (primary input from the main Avionics stack)
- 5V USB (auxiliary input for standalone operation and testing)

As well as a 24V input for powering the valve actuation circuits, which is stepped down to 8V4 using a boost converter when necessary to ensure stable operation of the ball valve actuation.

The *5V USB* input is intended for development, validation, and firmware flashing, allowing the board to operate independently of the full avionics stack.

An automatic power multiplexer is implemented to manage source selection. The circuit prioritizes the USB 5V input when both supplies are present. This prevents reverse current flow and mitigates the risk of damage due to improper or simultaneous connections.

Under nominal operating conditions, the board is powered from the 5 V Spine, supplied by the main Avionics stack through the spine connector.

A low-dropout regulator (LDO) then converts the selected 5 V input into a regulated 3.3V rail (3V3). This rail supplies the STM32 microcontroller, sensors, and digital logic components. The LDO is chosen for its low noise and stable output, ensuring reliable operation of sensitive electronics.

![power_rails_prc.png](/competition/Firehorn_2/Avionics/prc/power_rails_prc.png){.align-center}

## Storage
Onboard data logging is performed using a microSD card interfaced through a 4-bit SDIO bus.

The 4-bit SDIO interface enables high-throughput data transfers, ensuring reliable logging at the required 10 Hz acquisition rate without buffer overflows or data loss.

The SD card is dedicated to securely storing flight data, including sensor readings, power telemetry, and event logs. This allows for comprehensive post-flight analysis and performance evaluation in case the main avionics stack experiences a failure or data loss during flight.

## Debugging, flashing and monitoring
Firmware development and system validation are supported through:
- SWD (Serial Wire Debug) interface for programming and real-time debugging
- USB interface for firmware updates (DFU mode)
- Virtual COM Port (VCP) over USB for real-time telemetry and logging during testing

These interfaces allow developers to flash new firmware versions, set breakpoints, inspect memory and registers, and monitor system performance in real-time. The VCP provides a convenient channel for outputting debug information, sensor data, and status messages during development and testing.

## External communication
The PRC communicates with the main avionics stack through a spine connector using CAN bus for high-speed data exchange. CAN bus is chosen for its robustness, noise immunity, and suitability for real-time control applications in aerospace environments. The PRC can send telemetry data, receive commands, and synchronize with other avionics components over the CAN bus.

## Communication protocols recap
Here is a recap of all communication protocols used by the PRC for internal and external communication.

| Function | Communication Mode |
|---|---|
| Sensata (x5) (external sensor headers) | I²C (via PCA9547 mux, up to 8 channels) |
| FDC1004 (capacitance-to-digital) | I²C |
| INA228 (power monitor) | I²C |
| Kulite pressure transducer | Analogue — instrumentation amp (INA128) |
| PT1000 RTD sensors (×4) | Analogue — resistive |
| LMT85 (analogue temperature) | Analogue |
| microSD card | SDMMC |
| CAN bus (spine inter-board) | CAN FD (MCP2542) |
| USB DFU (J7) | OTG_USB_FS |
| USB VCP (J19) | OTG_USB_HS |
| SWD debug header (J27) | SWD |

# Physical Architecture
## Key Component Reference
|  Ref | Part | Package | Function | Notes |
|:-----:|:------:|:---------:|:----------:|:-------:|
| **U9** | [STM32H743VIT6](https://www.mouser.ch/ProductDetail/STMicroelectronics/STM32H743VIT6?qs=FNcb6ahWXRw0Me5k2ZHgWw%3D%3D) | LQFP-100 | Main microcontroller | ARM Cortex-M7, 2MB Flash, 1MB RAM. Central processor for all sensor acquisition, power control, actuation, communication, and data logging. |
| **Y1** | [16 MHz crystal](https://www.mouser.ch/ProductDetail/Murata/XRCGB16M000FXN14R0?qs=rrS6PyfT74fXIDf9XQeRjQ%3D%3D) | 0806 | MCU external clock source | Provides accurate clock reference for the STM32H743. Load capacitors C26/C27 (10pF) set the oscillator loop. |
| **U10** | [TPS2116DRL](https://www.mouser.ch/ProductDetail/Texas-Instruments/TPS2116DRLR?qs=aP1CjGhiNiGo3XAyXcVPaQ%3D%3D) | SOT-583-8 | Power multiplexer | Selects between USB 5V (TP13) and spine 5V (TP11) to supply the board's 5V rail, automatically prioritising the higher-voltage input. |
| **U5** | [MCP1727T-3302E/SN](https://www.mouser.ch/ProductDetail/Microchip/MCP1727T-3302E-SN?qs=usxtMOJb1Ry2jIwd9tEiSQ%3D%3D) | SOIC-8 | 3.3V LDO voltage regulator | Regulates 5V down to 3.3V for MCU and digital logic. |
| **IC4** | [MCP1642D-50I_MS](https://www.mouser.ch/ProductDetail/Microchip/MCP1642D-50I-MS?qs=kUzwwg2uGjVoXH8JFWZOdQ%3D%3D) | MCP1642 | 5V boost converter | Steps up a lower input voltage to 5V. Used to derive a regulated 5V rail from a possible power loss in cables. |
| **IC6** | [ADP1610ARMZ-R7](https://www.mouser.ch/ProductDetail/Analog-Devices-Inc/ADP1610ARMZ-R7?qs=WIvQP4zGanjTQwiTj85NkQ%3D%3D) | SOP-8 | Boost DC-DC converter | High-frequency boost converter used to generate an intermediate rail (11V). |
| **IC8** | [LTC3878EGN#PBF](https://www.mouser.ch/ProductDetail/Analog-Devices-Inc/LTC3878EGNPBF?qs=hVkxg5c3xu%2FW4MK%252B3jr5Vw%3D%3D) | SOP-16 | Synchronous buck controller | High-efficiency step-down controller for generating a high-current regulated at 8V4 from the 24V rail input. |
| **IC9** | [TPS25221DRVR](https://www.mouser.ch/ProductDetail/Texas-Instruments/TPS25221DRVR?qs=y6ZabgHbY%252BwMfKdJX5xA9w%3D%3D) | SON-7 | USB power switch with current limiting | Controls and protects the 5V USB power path. Provides overcurrent and reverse-current protection on USB supply input. |
| **IC1** | [INA228AIDGSR](https://www.mouser.ch/ProductDetail/Texas-Instruments/INA228AIDGSR?qs=pUKx8fyJudB2qGvt%252BmXPAg%3D%3D) | INA228 | Power/current monitor | High-precision voltage, current, and power measurement over I²C using shunt resistor R14 (13mΩ). Monitors the main supply rail for telemetry and protection. |
| **U3** | [INA128](https://www.mouser.ch/ProductDetail/Texas-Instruments/INA128UA-2K5?qs=VBduBm9rCJScYVe%252BDfqg5Q%3D%3D) | SOIC-8 | Instrumentation amplifier | High-precision differential amplifier for amplifying low-level sensor signals (e.g., Kulite pressure transducer or strain gauge). Drives ADC input of the MCU. |
| **U1, U11** | [MCP602](https://www.mouser.ch/ProductDetail/Microchip/MCP602-E-SN?qs=iRhCjdSJZe6hCMMETMefXQ%3D%3D) | SOIC-8 | Dual op-amp | General-purpose operational amplifiers used for signal conditioning on analogue sensor inputs. |
| **U4** | [LMT85DCK](https://www.mouser.ch/ProductDetail/Texas-Instruments/LMT85DCKR?qs=TB%2FQ0sBK%2FGeySHavof5VGg%3D%3D) | SOT-353 | Analogue temperature sensor | Outputs an analogue voltage proportional to temperature. Used for onboard thermal monitoring. |
| **U2** | [PCA9547PW](https://www.mouser.ch/ProductDetail/NXP/PCA9547PW118?qs=LOCUfHb8d9uNIvTa9zOc%252Bw%3D%3D) | TSSOP-24 | I²C multiplexer (8-channel) | Expands the MCU's I²C bus to 8 independent channels, allowing multiple sensors with the same address to coexist (e.g., Sens1–Sens5 headers). |
| **IC7** | [FDC1004DGSR](https://www.mouser.ch/ProductDetail/Texas-Instruments/FDC1004DGSR?qs=zEQ6BYqA5vF%252BEdgfKNFEaA%3D%3D) | SOP-10 | Capacitance-to-digital converter | Measures capacitance on up to 4 channels. Used for capacitive sensor acquisition (e.g., liquid level - FLS). Communicates over I²C. |
| **U6** | [MCP2542FDxMF](https://www.mouser.ch/ProductDetail/Microchip/MCP2542FD-E-SN?qs=qdgZG5p0FZ5pNizBl1u80Q%3D%3D) | SOIC-8 | CAN bus transceiver | Converts MCU CAN signals to differential CAN bus levels (CANH/CANL) for communication over the spine. Supports CAN FD. |
| **IC2, IC3, IC5** | [FAN3227TMX](https://www.mouser.ch/ProductDetail/onsemi/FAN3227TMX?qs=ODlIKnK5hgdsmwIkcdhHGg%3D%3D) | SOIC-8 | MOSFET gate drivers | Dual high-speed gate drivers for driving the power MOSFETs (Q2–Q5) in the valve actuation circuits. |
| **Q2, Q4** | [STS5DNF60L](https://www.mouser.ch/ProductDetail/STMicroelectronics/STS5DNF60L?qs=WSSg0AwdCh58Z5vYZGhZQg%3D%3D) | SOIC-8 | Dual N-channel MOSFET | Paired MOSFETs used for switching high-current loads such as motor/valve control. |
| **Q3, Q5** | [BSC093N04LSGATMA1](https://www.mouser.ch/ProductDetail/Infineon/BSC093N04LSGATMA1?qs=K00xGehIljvqz%2F%2Fk43Bdgg%3D%3D) | TDSON-8 | N-channel power MOSFET | High-current, low-Rds(on) MOSFETs used as synchronous rectifiers or low-side switches in the DC-DC buck converter stage. |
| **Q1** | [IRLML2502](https://www.mouser.ch/ProductDetail/Infineon/IRLML2502TRPBF?qs=9%252BKlkBgLFf37zQw9UlZd%2FQ%3D%3D) | SOT-23 | N-channel MOSFET | Small-signal low-side switch for enabling/disabling a peripheral or indicator. Driven by MCU GPIO. |
| **D14** | [1N5819](https://www.mouser.ch/ProductDetail/Diodes-Incorporated/1N5819HW-7-F?qs=NQ47qNm99eDyWTEd07miYA%3D%3D) | SOD-123 | Schottky diode | Reverse-polarity protection or OR-ing diode in the power path. Low forward voltage drop suits power switching applications. |
| **D18, D50** | [ZLLS1000TA](https://www.mouser.ch/ProductDetail/Diodes-Incorporated/ZLLS1000TA?qs=CUBnOrq4ZJx0h1V6PksUzA%3D%3D) | SOT-23-3 | Schottky diodes | Freewheeling or rectification diodes in switching converter or inductive load circuits. |
| **D1, D2, D6, D7, D38, D39** | [BAT54SW](https://www.mouser.ch/ProductDetail/Nexperia/BAT54SW115?qs=me8TqzrmIYUxdBsnXbLFqw%3D%3D) | SOT-323 | Dual Schottky diode array | Used for signal clamping, OR-ing, and protection on logic or low-power lines. |
| **D23, D26, D33, D34** | [1N4007](https://www.mouser.ch/ProductDetail/Rectron/1N4007W-T?qs=QNEnbhJQKvah3J4JbJr%2FPw%3D%3D) | SOD-123F | General-purpose rectifier diode | Freewheeling diodes across inductive loads (relays, solenoids, valve coils) to suppress back-EMF. |
| **D8** | [PESD4USB3UTBSQX](https://www.mouser.ch/ProductDetail/Nexperia/PESD4USB3UTBS-QX?qs=3Rah4i%252BhyCHIW05ia4xlKQ%3D%3D) | DFN-2510D-10 | USB TVS / ESD protection | Transient voltage suppressor on the data power input. Protects against overvoltage and ESD on the data lines. |
| **D3, D4, D5, D37, D40–D49, D57–D64** | [SP3021-01ETG](https://www.mouser.ch/ProductDetail/Littelfuse/SP3021-01ETG?qs=Dj8dEyEphkMM8WgFF2cfqw%3D%3D) | SOD-882 | 3.3V ESD protection diodes (×22) | Single-line ESD protection clamps on MCU GPIO, sensor signal lines, and communication interfaces. Clamping voltage 3.3V. |
| **D9, D29, D53, D54, D55** | [PESD5V0S1UL](https://www.mouser.ch/ProductDetail/Nexperia/PESD5V0S1UL-QYL?qs=rQFj71Wb1eWaKKPWHDNuZA%3D%3D) | SOD-882 | 5V ESD protection diodes (×5) | Single-line ESD clamps on 5V-tolerant lines such as USB, CAN, and power control signals. |
| **U12, U13** | [USBLC6-2P6](https://www.mouser.ch/ProductDetail/STMicroelectronics/USBLC6-2P6?qs=6ARB0lp6jlViGcbUSvj1Mw%3D%3D) | SOT-666 | USB ESD protection arrays (×2) | Dedicated USB ESD protection for the two USB Mini-B connectors (J7, J19), protecting D+/D− lines. |
| **F1** | [0805L100WR](https://www.mouser.ch/ProductDetail/Littelfuse/0805L100WR?qs=SJZ%252BTX%252BI2BTMlREVsW49pQ%3D%3D) | D_0805 | Resettable fuse (low current) | Overcurrent protection on a low-power rail or USB path. Resets after fault is cleared. |
| **F2** | [2920L600/24SLER](https://www.mouser.ch/ProductDetail/Littelfuse/2920L600-24SLER?qs=7MVldsJ5Uaz%2FccYHZvZIZA%3D%3D) | Fuse_2920 | Resettable fuse (high current) | Overcurrent protection on the main 24V or high-current supply input. Larger body for higher current rating. |
| **FB1** | [MPZ1608S121ATDH5](https://www.mouser.ch/ProductDetail/TDK/MPZ1608S121ATDH5?qs=pvNC7ksEVIcGR4VJPzSI2g%3D%3D) | 0603 | EMI filter / power decoupling | Ferrite bead used to suppress high-frequency noise on a supply rail, typically between digital and analogue power domains. |
| **L1, L2** | [744025004](https://www.mouser.ch/ProductDetail/Wurth-Elektronik/744025004?qs=rS3zZhy2AQMnR4D99KkS1Q%3D%3D) | WE-TPC_282892 | Switching converter inductors | Power inductors for DC-DC converter energy storage (boost or buck stages). |
| **L3** | [74437346047](https://www.mouser.ch/ProductDetail/Wurth-Elektronik/74437346047?qs=cFlH0mFHHFprMFXj0XGfyQ%3D%3D) | SRP7030CA2R2M | High-current switching inductor | Higher-current-rated power inductor for the main buck converter stage (IC8 / LTC3878). |
| **U7, U8** | [LTST-E133CEGBK](https://www.mouser.ch/ProductDetail/LITEON/LTST-E133CEGBK?qs=t7xnP681wgVl9mh9Y6AH%252BQ%3D%3D) | — | RGB status LED | RGB LED for additional status channels or redundant indication, controlled through PWM. |
| **BZ1** | [CMI-9605IC-0580T](https://www.mouser.ch/ProductDetail/Same-Sky/CMI-9605IC-0580T?qs=OlC7AqGiEDlyVLS7NTulfA%3D%3D) | TDK PS1240P02BT | Audible indicator | Provides audible alerts for system events such as arming, errors, or actuation confirmation. |
| **SW2** | [PCM12SMTR](https://www.mouser.ch/ProductDetail/CK-Switches/PCM12SMTR?qs=mfFuHy8STfL3qrPSfCHA7w%3D%3D) | SW_SPDT_PCM12 | Mode/configuration switch | Selects between two operating modes or configuration states (e.g., safe/armed, primary/backup). |
| **SW3** | [Omron B3U tactile switch](https://www.mouser.ch/ProductDetail/Omron/B3U-1100P?qs=q8ECNkb1%2FvMiNWwbhMFu4g%3D%3D) | SW_SPST_B3U-1000P | User pushbutton | Momentary input for user interaction such as arming confirmation, event logging, or reset. |
| **J9, J15** | [SPINE connector](https://www.digikey.ch/en/products/detail/stewart-connector/SS-12800-009/25879925) | Stewart SS-12800-009 | Spine serial input/output | Receives spine bus and passes it downstream to the next board in the chain. |
| **J7, J19** | [USB Mini-B](https://www.mouser.ch/ProductDetail/Amphenol/10033526-N3222MLF?qs=LmzVcvYPptRriY26yhJP4g%3D%3D) | Lumberg 2486_01 Horizontal | USB interfaces | Two USB Mini-B connectors for DFU firmware programming and virtual COM port (VCP) data access. |
| **J10** | [microSD holder](https://www.mouser.ch/ProductDetail/Hirose-Electric/DM3AT-SF-PEJM541?qs=iyLo5FA4poAlFf1J7h9Amg%3D%3D) | — | Storage interface | Accepts a microSD card for onboard data logging of sensor, power, and event data over SPI or SDMMC. |
| **J11** | [35312-0460](https://www.mouser.ch/ProductDetail/Molex/35312-0460?qs=K5bGVUuM57oIWvCwIBpyrQ%3D%3D) | — | Kulite pressure transducer input | 4-pin header for connecting a Kulite high-precision differential pressure sensor. Signal is amplified by INA128. |
| **J2, J3, J24, J25** | [501568-0207](https://www.mouser.ch/ProductDetail/Molex/501568-0207?qs=uEWtSLL707WbFjnit2JOEg%3D%3D) | — | PT1000 RTD temperature sensor inputs (x4) | Four connectors for PT1000 resistance temperature detectors. |
| **J1, J4, J5, J17, J23** | [87833-0419](https://www.mouser.ch/ProductDetail/Molex/87833-0419?qs=Vr3ZXvgdfhbIBPo%252BhQCZsA%3D%3D) | — | Generic sensor headers (×5) | Four-pin SMD headers for connecting external sensors routed through the I²C multiplexer. |
| **J6, J14, J16** | [53047-0210](https://www.mouser.ch/ProductDetail/Molex/53047-0210?qs=M5Ic86%252BuP8b8FQrTwriyOw%3D%3D) | — | External interface (x3) | General-purpose connector for external signals or expansion peripherals. |
| **J26** | [105314-2104](https://www.mouser.ch/ProductDetail/Molex/105314-2104?qs=5aG0NVq1C4z8s50hNvpgCw%3D%3D) | — | Ball valve actuator output | Connector for driving an electrically actuated ball valve. Controlled via MOSFET switching and protected by freewheeling diodes (D23/D26/D33/D34). |

## Step-Down Converter Design Justification (IC8 — LTC3878EGN#PBF)

The step-down stage is built around the LTC3878, a synchronous No R~SENSE~ valley current mode DC/DC controller operating from V~IN~ = 24V down to V~OUT~ = 8.4V, supplying the ball valve driver circuit and intermediate power rails. The output voltage is set by a resistive divider on the VFB pin according to the datasheet equation:

$$V_{OUT} = V_{REF} \times \left(1 + \frac{R52}{R28}\right) = 0.8 \times \left(1 + \frac{95k}{10k}\right) = 0.8 \times 10.5 = \mathbf{8.4\ V}$$

with V~REF~ = 0.8V (±1%) as specified in the LTC3878 datasheet, R52 = 95kΩ and R28 = 10kΩ.

**Switching frequency** is set by a resistor on the ION pin, which sources a current proportional to V~IN~ into the internal one-shot timer. The on-time is then:

$$t_{ON} = \frac{V_{OUT}}{V_{IN} \cdot f_{SW}}$$

The ION resistor R57 programs the on-time current, and the resulting frequency is compensated for V~IN~ variations to maintain good line stability — a key feature of the LTC3878 architecture.

**Inductor selection** (L3 = 4.7µH, SRP7030CA2R2M) is determined by the peak-to-peak ripple current target, typically 20–40% of maximum load current. The inductor ripple current is given by:

$$\Delta I_L = \frac{V_{OUT}}{L \cdot f_{SW}} \times \left(1 - \frac{V_{OUT}}{V_{IN}}\right) = \frac{8.4}{4.7\times10^{-6} \cdot f_{SW}} \times \left(1 - \frac{8.4}{24}\right)$$

The duty cycle at nominal input is D = V~OUT~/V~IN~ = 8.4/24 = 0.35 (35%), well within the LTC3878's wide operating range. The SRP7030 package is selected for its high saturation current rating and low DCR, minimising conduction losses at the expected load currents without a separate sense resistor — consistent with the LTC3878's No R~SENSE~ architecture, which uses the top MOSFET V~DS~ for current sensing.

**Output capacitance** is provided by a bank of 4× 22µF ceramic capacitors (C45–C47, C52) in parallel with a 100µF electrolytic (C51), yielding approximately 188µF total. The ceramic capacitors (0402, low ESR) dominate the high-frequency ripple filtering. Output voltage ripple in steady state is primarily determined by the capacitor ESR and the inductor ripple current:

$$\Delta V_{OUT} \approx \Delta I_L \times ESR_{C_{OUT}}$$

The large total capacitance also ensures a controlled output voltage undershoot during fast load transients, which is important given the inductive ball valve load on J26. The LTC3878 is specifically noted in its datasheet to be stable with low ESR ceramic output capacitors, making this all-ceramic-plus-electrolytic combination appropriate.

**Input capacitance** is symmetrically sized at 4× 22µF ceramic (C41, C48, C49, C53) plus 100µF electrolytic (C42), also ~188µF. The input capacitors absorb the pulsed input current drawn during the top MOSFET on-time. The required RMS current rating is:

$$I_{C_{IN}(RMS)} \approx I_{OUT} \cdot \sqrt{D \cdot (1 - D)} = I_{OUT} \cdot \sqrt{0.35 \times 0.65} \approx 0.48 \cdot I_{OUT}$$

The distributed ceramic bank reduces the effective ESR seen at the switching node, limiting V~IN~ ripple and switching noise injected back onto the 24V supply rail.

**The two BSC093N04LSGATMA1 N-channel MOSFETs** (Q3 top, Q5 bottom) are driven by IC8's TG and BG outputs respectively. Their low R~DS(ON)~ minimises conduction losses at the high step-down ratio (35% duty cycle means the bottom MOSFET conducts 65% of the switching period, making bottom switch conduction loss the dominant term). The bootstrap capacitor C43 (100nF) and Schottky diode D18 supply the floating gate drive voltage for the top MOSFET via the BOOST pin, as required by the LTC3878's high-side driver architecture.

**Loop compensation** is set by R27 (20kΩ) and C38 (100nF) with C39 (100pF) in parallel on the ITH pin. This RC network shapes the error amplifier response, setting the crossover frequency and phase margin of the voltage control loop. As noted in the datasheet, the crossover frequency should remain below 20% of the switching frequency to ensure stable operation.

![step_down.jpg](/competition/Firehorn_2/Avionics/prc/step_down.jpg){.align-center}

## Technical Drawing

### Schematic
![schematic_prc_stm32.jpg](/competition/Firehorn_2/Avionics/prc/schematic_prc_stm32.jpg){.align-center}
![schematic_prc_spine&power.jpg](/competition/Firehorn_2/Avionics/prc/schematic_prc_spine&power.jpg){.align-center}
![schematic_prc_sensors.jpg](/competition/Firehorn_2/Avionics/prc/schematic_prc_sensors.jpg){.align-center}
![schematic_prc_out.jpg](/competition/Firehorn_2/Avionics/prc/schematic_prc_out.jpg){.align-center}

### Layouts
![layout_prc_f.jpg](/competition/Firehorn_2/Avionics/prc/layout_prc_f.jpg){.align-center}
![layout_prc_ln1.jpg](/competition/Firehorn_2/Avionics/prc/layout_prc_ln1.jpg){.align-center}
![layout_prc_ln2.jpg](/competition/Firehorn_2/Avionics/prc/layout_prc_ln2.jpg){.align-center}
![layout_prc_f.jpg](/competition/Firehorn_2/Avionics/prc/layout_prc_f.jpg){.align-center}



### Possible Improvements



# Technical Budget, Margins and Deviation

| Type of value  | Units | Requirement Value  | Actual Value | Deviation |
|----------------|-------|:------------------:|:------------:|:---------:|
| Dimensions   | mm    | Smallest form factor possible | 115.23 x 50 x 20           |  0 %  |
| Weight   | g    | 250                 | ?           | ?  |

# Design Constraints
## Constraints for Operation
The PRC must operate reliably in the expected environmental conditions of a rocket flight, including:
- Temperature range: -80°C to +85°C
- Vibration: up to 5 g RMS across 20–2000 Hz
- Lift-off shock: up to 100 g for 100 ms
- Humidity: up to 95% condensing
- Freezing: LOX tank environment

The board must also be operable in tightly constrained spaces within the rocket bays, with limited access for maintenance or adjustments once integrated. This necessitates a compact form factor and robust connectors to ensure reliable operation throughout the mission profile. As well as being considerate of connector placement and orientation due to surrounding plumbing and structural elements in the rocket design, which may limit access to certain ports or require specific cable routing.