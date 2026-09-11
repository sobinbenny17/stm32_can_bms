# CAN-Bus Battery Management System with Simulink-Based HIL Validation

A Battery Management System (BMS) built on an STM32F4 Discovery board, validated using a Hardware-in-the-Loop (HIL) test bench instead of a real battery pack. A Simulink-derived battery model runs on an Arduino + MCP2515 node and broadcasts realistic, evolving battery telemetry over a real CAN bus. The STM32 listens to this traffic, estimates State of Charge (SOC) using an Extended Kalman Filter (EKF), and applies real-time safety protection logic — exactly as a production BMS would.

## Why this project exists

Real EVs and e-bikes use a BMS that communicates over CAN bus, the standard protocol used across almost every vehicle and industrial machine on the road today. Instead of buying real battery cells, this project builds a mathematical model of a battery in MATLAB/Simulink and runs it on hardware that speaks real CAN messages — a genuine, industry-standard technique (HIL testing) used by automotive and industrial companies to validate ECUs before ever touching physical batteries or engines.

This project demonstrates three things:
1. Accurately **modeling a physical system** (a battery) in Simulink.
2. Writing **embedded firmware** that communicates over CAN and makes real-time protection decisions.
3. Building and validating a **HIL test bench** that proves the firmware works against a simulated plant, not just a software mock.

## System architecture

| Node | Hardware | Role |
|---|---|---|
| **Plant (simulated battery)** | Arduino Uno + MCP2515 CAN module | Continuously computes what a real battery's voltage/current would be doing (via the Simulink-derived model) and broadcasts it as CAN frames, standing in for real battery sensors |
| **ECU (the BMS)** | STM32F406RGT6 Discovery board + SN65HVD230/TJA1050 CAN transceiver | Listens to simulated battery data over CAN, runs SOC estimation (EKF) and protection logic, and broadcasts its own status/fault messages |
| **Bus** | Real two-wire CAN_H/CAN_L, 120Ω termination at each end | Genuine electrical CAN traffic — real bus timing, arbitration, and electrical behavior, not a simulation |

## Hardware

- STM32F406RGT6 Discovery board — runs the BMS firmware
- SN65HVD230 or TJA1050 CAN transceiver module
- Arduino Uno — runs the plant/battery simulator firmware
- MCP2515 CAN module (with onboard TJA1050 transceiver)
- 2× 120Ω resistors — bus termination
- Breadboard + jumper wires
- *(Optional)* I2C 16x2 LCD/OLED — live BMS status display
- *(Optional)* USB-CAN adapter — live bus sniffing/logging on PC

## Software / tools

- **STM32CubeIDE** — BMS firmware
- **Arduino IDE** — plant simulator firmware (`mcp_can` library by coryjfowler for the MCP2515)
- **MATLAB + Simulink** with Vehicle Network Toolbox — battery modeling and CAN message prototyping
- *(Optional)* `python-can` + USB-CAN adapter — live bus debugging

## Project sections

1. **CAN fundamentals + STM32 loopback proof** — validate the STM32's CAN peripheral, clock config, and bit-timing internally, with no external hardware.
2. **Battery modeling in Simulink** — 1st-order Thevenin equivalent-circuit model (OCV-SOC curve, R0, one RC branch) built from published Panasonic 18650PF cell data.
3. **Extended Kalman Filter for SOC estimation** — replaces naive coulomb counting with an EKF that fuses the Thevenin model with noisy voltage/current measurements.
4. **Real 2-node CAN network + STM32 BMS firmware** — move off loopback onto a physical bus; STM32 parses live battery data, runs the ported EKF and protection logic, and broadcasts BMS status/fault frames.
5. **Arduino + MCP2515 plant simulator node** — the discretized battery model runs standalone on the Arduino, broadcasting realistic telemetry independent of any PC.
6. **Full integration + fault injection demo** — inject fault conditions (overvoltage, overcurrent, disconnect) from the Arduino side and confirm the STM32 BMS responds correctly in real time.

## Status

This project is in progress. See the section breakdown above for what's implemented so far. (Update this line as sections are completed.)

## Repository structure

```
├── Core/              # STM32 application source (Src/Inc) and startup code
├── Drivers/            # STM32 HAL and CMSIS drivers
├── *.ioc               # STM32CubeMX pin/peripheral configuration
├── *.ld                # Linker script
└── README.md
```

## Getting started

1. Clone this repository.
2. Open **STM32CubeIDE** → File → Import → Existing Projects into Workspace → select this folder.
3. Build and flash to an STM32F406RGT6 Discovery board.
4. Wire the CAN transceiver and Arduino/MCP2515 node as described in the architecture section above, with 120Ω termination at each bus end.
5. Flash the Arduino plant-simulator sketch *(link/location to be added once committed)*.
6. Power both nodes and observe BMS SOC estimation and fault handling over the live CAN bus.

## License

*(Add a license, e.g. MIT, if you'd like others to freely use/reference this project.)*
