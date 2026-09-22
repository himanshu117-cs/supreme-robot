## Table of Contents

1. [Overview]
2. [Key Features]
3. [System Architecture]
4. [Repository Structure]
5. [Hardware Summary]
. [Electronics & Control]
7. [Getting Started]
8. [Prerequisites]
9. [Performance Estimates]
10. [Known Limitations & Roadmap]
11. [Safety Warnings]
12. [Contributing]
13. [Acknowledgments]



## Overview

**Foreamr V1.2** (forearm) and **Robo Hand.2** (hand) together form a self-contained robotic prosthetic/assistive forearm. The hand is driven by five underactuated, servo-linked digits capable of preset grasp shapes, controlled through an offline voice-recognition module and a touchscreen HMI, with infrared contact sensing that lets the hand stop closing on an object rather than crushing it or stalling against it.

The entire structure is additively manufactured (FDM 3D-printed) and assembled with brass heat-set inserts and M3 socket-head cap screws, making it inexpensive to build, modify, and repair compared to commercial myoelectric prostheses. All electronics — microcontroller, servo driver, battery, and wiring — are enclosed within the forearm shell, with the hand rotating freely on a bearing-supported wrist joint.

This repository contains the complete engineering documentation and firmware for the project: a full IEEE-format design paper, working ESP32 firmware for every subsystem, and calibration/training utilities needed to bring a physical build online.

> **Project status:** mechanical design complete (CAD), firmware complete and un-flight-tested on hardware, engineering estimates pending validation against a physical prototype.


## Key Features

- **5-digit underactuated hand** — one MG996R servo per digit drives a multi-bar linkage that curls multiple joints from a single input
- **Voice control** — offline, trainable voice recognition (Elechouse VR3) triggers preset grasp gestures with no cloud dependency
- **Touchscreen HMI** — Nextion 2.4" display for status, battery level, and manual gesture selection as a voice fallback
- **Adaptive grasp** — five IR contact sensors let each digit independently stop closing on contact, producing an object-conforming grip from open-loop servos
- **Centralized control** — a single ESP32 coordinates actuation, sensing, HMI, and power monitoring over I²C/UART
- **Serviceable construction** — fully 3D-printed shells fastened with brass heat-set inserts, designed for repeated disassembly
- **Complete firmware stack** — non-blocking state machine, slew-rate-limited motion, staggered actuation to bound current draw, and low-battery lockout


## System Architecture

A single ESP32 arbitrates three input sources — voice, touchscreen, and per-digit contact sensors — and drives all actuation through a PCA9685 16-channel PWM controller. Power is delivered from a 2S lithium pack through a step-down converter sized for simultaneous multi-servo current draw.
The hand attaches to the forearm through a bearing-supported rotary joint: a shaft on the palm base plate is radially supported by two flanged bearings seated in the forearm shell, giving the wrist a free rotational degree of freedom while wiring passes through the bore to the electronics bay.


## Repository Structure
.
├── README.md                            ← you are here
├── Prosthetic_Arm_IEEE_Paper.docx       ← full engineering paper (IEEE format)
│
├── assets/                              ← renders and diagrams used 
│   ├── render.png
│   ├── hand closeup.png
│   ├── system block diagram.png
│   └── wrist joint diagram.png
│
└── ProstheticArm_Firmware/              ← ESP32 firmware (Arduino)
    ├── ProstheticArm_Firmware.ino       ← main sketch: state machine, adaptive grasp
    ├── config.h                         ← all pins, calibration, tuning constants
    ├── ServoController.h                ← PCA9685 driver, slew limiting, stagger
    ├── GestureLibrary.h                 ← preset poses, voice/touch → gesture map
    ├── ContactSensors.h                 ← debounced LM393 reading, edge detection
    ├── VoiceInput.h                     ← VR3 protocol driver (HardwareSerial)
    ├── NextionHMI.h                     ← Nextion serial protocol + touch events
    ├── PowerMonitor.h                   ← battery voltage, SoC, low-voltage lockout
    ├── README.md                        ← wiring table + firmware bring-up guide
    └── tools/
        ├── ServoCalibration/            ← interactive servo endpoint finder
        └── VoiceTraining/               ← interactive VR3 word trainer


## Hardware Summary

Subsystem Part Notes
Compute ESP32 dev board with expansion board Main controller, WiFi/BT-capable MCU
Actuation PCA9685 16-channel PWM controller driving 5x MG996R RC servo One servo per digit, controlled over I2C bus
Sensing 5x LM393 IR comparator module Contact/proximity sensing for each digit
Display Nextion 2.4" touch screen Status display and manual input
Voice Elechouse Voice Recognition V3 module Speaker-dependent command recognition (up to 7 slots)
Power 2S 7.4V 1500mAh LiPo + 9A/300W buck converter Regulated power for servo array
Structural FDM 3D printed shells, M3 DIN 912 screws and brass heat-set inserts Fully serviceable enclosure
Wrist joint 2x flanged bearing + 2x sleeve bushing Passive rotational joints


## Electronics & Control

The system is equipped with multiple interfaces for peripheral control and supervision. The I²C bus is used to communicate with the PCA9685 servo driver and send position orders to make the fingers move. UART1 is used to interface with the Nextion display to show the status of the system and manage touch events, while UART2 is linked to the Voice Recognition V3 module to get speech-recognized commands as character data. Five GPIO pins are linked to LM393 sensor modules to detect contact between each of the five fingers and the keypad plate. Finally, the ADC is connected to a battery voltage-divider to observe the battery-pack voltage.


## Getting Started

1. **Print and assemble** the mechanical structure per the CAD design; install brass heat-set inserts with a soldering iron before final assembly.
2. **Wire the electronics** following the wiring table in the firmware README — pay particular attention to the shared ground and the bulk capacitor across the servo power rail.
3. **Calibrate the servos** — this finds safe pulse-width endpoints and prevents stalling a servo against a mechanical limit.
4. **Train the voice module** — a one-time, speaker-dependent procedure.
5. **Build the Nextion HMI project** in Nextion Editor per the component map in the firmware README, and flash it to the display.
6. **Flash** to the ESP32 and power on.


### Prerequisites

- Arduino IDE with the **esp32 by Espressif Systems** board package 
- **Adafruit PWM Servo Driver** library 
- Nextion Editor 
- A multimeter — required for both battery divider calibration and safe buck-converter setup


## Performance Estimates

The table below summarizes the key characteristics of our design.
Metric Estimate Basis
Actuated DOF 4-5 3 fingers + thumb flexion (+opposition if independently actuated)
Wrist DOF 1, passive bearing-supported, manually positioned
Servo torque 1 (per digit, 6V) ~1.08 N·m stall / ~0.86 N·m usable MG996R datasheet
Fingertip force (per ~29 N (est.) 30 mm effective moment arm assumption
digit)
Theoretical grip payload ~4.8 kgf (upper bound) friction limited, μ≈0.55
Practical working payload 0.5 - 2 kg (recommended) joint strength (printed plastic) limited, not servo torque
Peak current draw (multi- 6 - 9 A realistic, 15 A worst case confirms 9A/300W converter sizing
servo)
Battery runtime 3 hours (intermittent use) 1500 mAh pack, ~500 mA average draw



## Known Limitations & Roadmap

Identified gaps in the current design, in order of priority:
[x] No biosignal input — control is voice/touch based, not EMG/IMU based; this is an assistive robotic hand, not a myoelectric prosthesis, unless this is added
[x] No battery management/protection circuit — the 2S pack has nothing in the way of discharge/current protection or cell balancing (hardware-wise); firmware-based voltage monitoring is insufficient
[x] Passive wrist — no motor in the wrist joint; PCA9685 channel 5 is set aside in the firmware for this
[x] Binary contact sensing only — cannot apply variable grip force, only fully close or open; would need FSRs or current sensing amps (like INA219) on each servo to do so
[x] Unverified performance estimates — torque, payload, and runtime are all theoretical and should be stress-tested on an actual build


## Safety Warnings

Servo stall risk: an MG996R driven beyond the linkage's mechanical limit stalls at approximately 2.5 A and can destroy its gear train within one minute of continuous current. Servos must be calibrated (run through their full range) before installing linkages and must be recalibrated after any mechanical adjustment.
Battery safety: this design contains no battery management system at all. Do not leave the pack connected to a charger unattended, and consider adding a protection module to the system.
This is not a certified medical device, and it should not be used as one. If you intend to use this as a personal prosthesis, consult with an appropriate prosthetics professional and familiarize yourself with applicable medical device regulations before proceeding.


## Contributing

Systems integration: marrying together three individually-known concepts (voice control, underactuated linkage fingers, IR-based adaptive grasp) into one functional whole.
Low-cost adaptive grasp: using a low priced/cheap IR LED/phototransistor pair and some freeze-on-contact firmware to emulate the feel of a high-end prosthetic hand with force-sensitive resistors on every finger.
Demonstrated reproducibility: full Bill of Materials, wiring diagrams, working firmware, and calibration methodology provided in the hope that others might build upon this project, beyond the typical instructable-style webpage with a video and parts list.
A cost-benefit case study for a multi-DOF, force- and voice-controlled hobbyist myolectric hand, built from consumer components, as an alternative to commercial prosthetics.


## Acknowledgments

Built around the Espressif ESP32, Adafruit's PCA9685 driver library, the Elechouse Voice Recognition V3 module, and Nextion's HMI display line. See the design paper's References section for datasheet citations.

