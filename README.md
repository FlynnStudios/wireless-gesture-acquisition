# Wireless Gesture Acquisition & Unity Visualization

<p align="center">
  <img src="images/hero.jpg" width="760" alt="Wireless gesture acquisition system demonstration">
</p>

A five-channel finger-motion acquisition system that converts physical finger movement into analog sensor signals, digitizes them using dual ADS1115 ADCs, processes the measurements on an STM32F103C8T6, and transmits the resulting data wirelessly for PC-side visualization.

The project integrates physical sensing, embedded data acquisition, Bluetooth serial communication, PC-side data transfer, virtual-hand visualization, and sensor-data recording into a complete end-to-end prototype.

**Project period:** Jan 2023 – Nov 2023  
**Institution:** Jinan University  
**Project type:** Undergraduate Thesis

---

## Project Goal

The project explored a low-cost approach to acquiring finger-motion data for human-machine interaction.

Instead of relying on camera-based tracking or a high-degree-of-freedom sensor glove, the system uses one linear potentiometer for each finger to convert physical displacement into an analog voltage.

The five sensing channels are digitized, processed by an embedded controller, transmitted wirelessly, and mapped to a virtual hand for live visualization.

The goal was therefore not to build a machine-learning gesture classifier, but to develop a complete **finger-motion acquisition and visualization pipeline** from physical sensing to PC-side presentation.

---

## Highlights

- **5 independent finger-motion sensing channels**
- **5 × linear potentiometers** for finger displacement sensing
- **2 × ADS1115 16-bit ADCs** for five-channel analog acquisition
- `STM32F103C8T6` embedded controller
- Two independent software-I2C buses
- HC-05 Bluetooth serial communication
- Five-channel ASCII packet transmission at **115200 baud**
- PC-side serial-to-TCP data relay
- Unity-based virtual-hand visualization
- Sensor-data recording and `.xls` export
- Complete physical prototype with live motion demonstration

---

## My Contributions

My work focused primarily on the sensing, embedded acquisition, wireless communication, and overall system-integration portions of the project.

- Developed the overall five-channel finger-motion acquisition concept.
- Built and integrated the sensing mechanism using one linear potentiometer per finger.
- Integrated two ADS1115 ADC modules with an `STM32F103C8T6` for five-channel analog acquisition.
- Implemented the application-level STM32 acquisition logic, channel sequencing, relative-value scaling, packet formatting, and UART transmission.
- Integrated two software-I2C buses for communication with the ADC modules.
- Configured and integrated the paired HC-05 Bluetooth serial link between the acquisition device and the PC side.
- Defined the sensing-to-visualization system concept and integrated the embedded data path with the PC-side visualization workflow.
- Tested the complete system through live finger-motion demonstrations and recorded acquisition data.

> **Contribution boundary:** The PC-side C# / Unity visualization implementation was adapted from existing code rather than written entirely from scratch. My contribution on the visualization side focused on the system concept, data integration, configuration, and end-to-end operation rather than independent C# software development.

---

## System Architecture

<p align="center">
  <img src="hardware/system-architecture.png" width="820" alt="Wireless gesture acquisition system architecture">
</p>

The system follows a complete sensing-to-visualization data path:

```text
Finger Motion
      ↓
5 × Linear Potentiometers
      ↓
2 × ADS1115 ADCs
      ↓
STM32F103C8T6
      ↓
HC-05 Bluetooth Link
      ↓
PC Serial / TCP Relay
      ↓
Unity Visualization
```

The potentiometers convert physical finger displacement into analog voltages. The ADS1115 modules digitize the signals, the STM32 processes and packages the five measurements, and the resulting data is transmitted wirelessly to the PC-side visualization pipeline.

---

## Physical Sensing & Embedded Acquisition

Each finger is mechanically coupled to one linear potentiometer.

As the finger moves, the potentiometer slider changes position and produces an analog voltage associated with that finger's displacement.

<p align="center">
  <img src="images/sensor-prototyping.jpg" width="620" alt="Finger sensor mechanism prototyping">
</p>

This creates five independent sensing channels while intentionally representing each finger with **one overall motion value** rather than attempting to measure every individual finger joint.

### Acquisition Hardware

<p align="center">
  <img src="hardware/embedded-acquisition-flow.png" width="820" alt="Embedded five-channel acquisition flow">
</p>

| Function | Implementation |
| --- | --- |
| Finger sensing | 5 × linear potentiometers |
| ADC | 2 × ADS1115, 16-bit |
| ADC #1 | Four sensing channels |
| ADC #2 | One sensing channel |
| MCU | STM32F103C8T6 |
| Software I2C #1 | PB6 / PB7 |
| Software I2C #2 | PA4 / PA5 |
| Wireless link | Paired HC-05 modules |
| UART rate | 115200 baud |

The two ADS1115 modules use separate software-I2C buses. In the preserved implementation, both ADC modules use the same 7-bit I2C address while remaining independent because they are connected to different buses.

### Controller Hardware

<p align="center">
  <img src="images/controller-board.jpg" width="560" alt="STM32 gesture acquisition controller board">
</p>

The prototype controller combines the STM32, dual ADS1115 modules, HC-05 Bluetooth module, sensor connections, and supporting prototype wiring.

### Hardware Integration

<p align="center">
  <img src="images/hardware-integration.jpg" width="560" alt="Controller integrated into gesture acquisition device">
</p>

The controller was integrated directly into the physical sensing assembly, creating a complete acquisition device from mechanical finger input through embedded processing and wireless transmission.

---

## Sensor Scaling & Data Packet

Absolute ADC measurements are not ideal for describing finger state because the usable sensor travel depends on the physical sensing mechanism and user geometry.

The preserved firmware converts the measured values into a relative finger-state representation using a shared startup range:

```text
Establish startup range
        ↓
Read ADC measurement
        ↓
Apply shared range scaling
        ↓
Relative finger value
(nominal 0–100)
```

> **Implementation note:** The thesis described a more general travel-calibration approach intended to compensate for differences between users and fingers. The preserved firmware version uses a simpler shared startup-range implementation rather than independent minimum/maximum calibration parameters for all five channels.

### Data Packet Format

The STM32 packages the latest five finger values into a simple ASCII frame:

```text
AT+F1+F2+F3+F4+F5+ED
```

Each finger value is represented as a three-digit decimal field.

Example:

```text
AT+020+056+060+090+050+ED
```

Conceptually:

```text
AT
 │
 ├── +020   Finger 1
 ├── +056   Finger 2
 ├── +060   Finger 3
 ├── +090   Finger 4
 ├── +050   Finger 5
 │
 ED
```

The preserved firmware transmits the packet over UART at **115200 baud**.

The text-based format also made the data easy to inspect during development and straightforward to parse on the PC side.

---

## Wireless & PC Data Path

Two HC-05 modules were configured as a paired serial connection between the embedded device and the PC side.

```text
STM32
  ↓ UART
HC-05
(Device Side)
  ↓
Bluetooth
  ↓
HC-05
(PC Side)
  ↓
Serial / COM
```

The PC receives the Bluetooth data as a serial COM stream.

A lightweight C# bridge then forwards the raw serial stream to a local TCP connection:

```text
Serial / COM
      ↓
C# Serial–TCP Bridge
      ↓
TCP localhost:8086
      ↓
Unity Application
```

The bridge acts primarily as a transport layer. It does not perform the virtual-hand mapping itself; frame parsing and visualization occur downstream in Unity.

---

## Unity Visualization

<p align="center">
  <img src="images/unity-visualization.png" width="760" alt="Unity virtual hand visualization">
</p>

The Unity application displays a virtual hand whose five fingers respond independently to the incoming sensor values.

The visualization does **not** perform full skeletal pose estimation, inverse kinematics, or machine-learning gesture recognition.

Instead, five finger animations were prepared in advance between extended and flexed states. The incoming relative finger value selects a position within the corresponding animation.

```text
Relative Finger Value
        ↓
Select Finger Animation
        ↓
Set Animation Playback Position
        ↓
Virtual Finger State
```

### Animation Setup

<p align="center">
  <img src="images/unity-setup.png" width="700" alt="Unity finger animation setup">
</p>

The visualization uses Unity animation `normalizedTime` to select the displayed state of each finger animation.

This provides an intuitive representation of the five-channel sensor data without requiring a full hand-kinematics model.

The Unity-side application also supports recording the received finger-state data and exporting it to an `.xls` file for later analysis or reuse.

---

## Demo & Results

[▶ Watch the gesture-acquisition demo](demo/gesture-demo.mp4)

The demonstration shows physical finger motion on the acquisition device being reflected by the virtual hand on the PC.

The completed system demonstrated that five independent finger-motion channels could be:

- physically sensed;
- digitized through the ADS1115 modules;
- processed and packaged by the STM32;
- transmitted over the Bluetooth serial link;
- relayed to the PC application;
- visualized as independent virtual-finger states;
- recorded for later use.

The original thesis demonstration included several example hand configurations, including:

- pinch;
- a "six" hand gesture;
- "LOVE";
- a closed-hand gesture.

These examples were used to demonstrate the behavior of the acquisition pipeline rather than to indicate a gesture-classification algorithm.

---

## Engineering Limitations & Lessons

### 1. Five-Channel Representation

Each finger is represented by one overall motion value.

Because a real finger contains multiple independently moving joints, the system cannot reconstruct detailed joint-level hand pose.

A higher-resolution design could use multiple sensing elements per finger to capture additional degrees of freedom.

### 2. Animation-Based Visualization

The Unity side maps each sensor value to a position within a predefined finger animation rather than directly controlling individual skeletal joints.

This simplifies visualization but limits the flexibility and fidelity of the virtual-hand representation.

A future implementation could map calibrated sensor values directly to individual joint rotations.

### 3. Shared Startup Scaling

The preserved firmware uses one shared startup-range scaling approach rather than maintaining independent calibration parameters for all five sensing channels.

Independent per-channel calibration would better account for differences between individual fingers, sensor mounting, and users.

### 4. Sequential Acquisition

The preserved firmware updates the sensing channels sequentially rather than using a timer-driven or buffered acquisition architecture.

A future implementation could use scheduled sampling, structured data buffers, explicit range checking, and per-channel calibration to improve timing consistency and robustness.

---

## Repository Scope

This repository is intended as an **engineering portfolio and technical case study**, not as a complete open-source hardware or software release.

The following materials are intentionally not published:

- the complete original STM32 firmware project;
- third-party STM32 peripheral-library code;
- third-party software-I2C driver implementations;
- the complete Unity project;
- C# visualization code adapted from existing implementations;
- the full undergraduate-thesis development archive;
- external reference materials and third-party libraries.

The published materials are intended to document the physical sensing architecture, embedded acquisition pipeline, wireless communication, system integration, testing, and final prototype.

---

## Acknowledgments

This project was developed as an undergraduate thesis project at **Jinan University**.

I am grateful to my thesis advisor and to the authors of the external software resources used during the PC-side visualization development.
