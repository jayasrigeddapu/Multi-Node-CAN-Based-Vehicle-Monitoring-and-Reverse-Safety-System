# Multi-Node CAN-Based Vehicle Monitoring and Reverse Safety System

## 📌 Project Overview

The **Multi-Node CAN-Based Vehicle Monitoring and Reverse Safety System** is an embedded systems project designed to improve vehicle monitoring and reverse safety using the **Controller Area Network (CAN) protocol**.

The system uses an LPC2129 microcontroller, a DS18B20 temperature sensor, an HC-SR05 ultrasonic sensor, and multiple CAN-connected nodes to monitor engine temperature, control vehicle indicators, and detect obstacles during reverse operation.

It provides visual and audible alerts to help the driver identify obstacles while reversing.

## 🎯 Objectives

- Monitor engine temperature using the DS18B20 temperature sensor.
- Control vehicle indicator signals through CAN communication.
- Detect obstacles behind the vehicle using an ultrasonic sensor.
- Generate buzzer or LED alerts when an obstacle is detected during reverse mode.
- Establish communication between multiple embedded nodes using the CAN protocol.

## ⚙️ System Architecture

The system consists of three main nodes:

### 1. Main Node

- Reads engine temperature using the DS18B20 sensor.
- Displays temperature information on the LCD.
- Detects switch inputs through external interrupts.
- Sends indicator control commands to the Indicator Node through CAN.
- Switches between forward and reverse operating modes.
- Receives reverse-alert information and activates a buzzer or LED when necessary.

### 2. Indicator Node

- Monitors CAN messages from the Main Node.
- Receives indicator control commands.
- Controls the corresponding indicator LEDs based on the received data.

### 3. Reverse Alert Node

- Reads distance measurements from the HC-SR05 ultrasonic sensor.
- Compares the measured distance with a configured threshold.
- Sends logic `1` to the Main Node when the distance is below the threshold.
- Sends logic `0` when the distance is above the threshold.

## 🧰 Hardware Requirements

- LPC2129 Microcontroller
- MCP2551 CAN Transceiver
- DS18B20 Temperature Sensor
- HC-SR05 Ultrasonic Sensor
- LCD Display
- LEDs
- Push Buttons / Switches
- USB-to-UART Converter
- Buzzer for audible alerts

## 💻 Software Requirements

- Embedded C
- Keil C Compiler
- Flash Magic
- UART communication tools, if required for debugging

## 🔄 Working Principle

1. The Main Node continuously reads engine temperature and displays it on the LCD.
2. When indicator switches are activated, the Main Node sends the appropriate control information through CAN.
3. The Indicator Node receives CAN messages and controls the indicator LEDs.
4. A mode switch changes the system between forward and reverse modes.
5. During reverse mode, the Reverse Alert Node measures the distance to an obstacle using the ultrasonic sensor.
6. The measured distance is compared with a predefined threshold.
7. The Reverse Alert Node sends the corresponding alert status to the Main Node through CAN.
8. The Main Node activates a buzzer or LED when an obstacle alert is received.

## 🛠️ Implementation Procedure

1. Create the project folder and configure the development environment.
2. Test the LCD by displaying characters, strings, and integer values.
3. Interface the DS18B20 sensor and display the engine temperature.
4. Test external interrupts using switches and display the interrupt count.
5. Interface the HC-SR05 ultrasonic sensor and verify distance measurements.
6. Develop and test basic CAN communication on the hardware.
7. Implement the Main Node, Indicator Node, and Reverse Alert Node separately.
8. Integrate the nodes and test communication, indicator control, operating modes, and reverse obstacle alerts.

## 📚 Key Technical Concepts

- Embedded C Programming
- LPC2129 Architecture
- CAN Bus Communication
- GPIO Interfacing
- External Interrupts
- Temperature Sensor Interfacing
- Ultrasonic Distance Measurement
- LCD Interfacing
- Multi-Node Embedded System Design

## 🚗 Applications

- Vehicle indicator control systems
- Reverse parking assistance
- Embedded vehicle monitoring
- CAN-based automotive communication
- Educational automotive embedded systems projects

## 🔮 Future Enhancements

- Add an LCD distance display for reverse parking.
- Implement multiple obstacle-distance warning levels.
- Add data logging for engine temperature and obstacle events.
- Integrate additional CAN nodes for other vehicle monitoring functions.
- Extend the prototype with automotive-grade hardware and validation.

## 👨‍💻 Project Information

- **Project Name:** Multi-Node CAN-Based Vehicle Monitoring and Reverse Safety System
- **Microcontroller:** LPC2129
- **Programming Language:** Embedded C
- **Communication Protocol:** CAN
- **Development Tools:** Keil C Compiler and Flash Magic
- **Project Domain:** Embedded Systems and Automotive Electronics

## 📄 Note

This README describes the system design. Actual implementation results depend on hardware connections, CAN configuration, sensor calibration, and firmware testing. Add verified output screenshots and the project block diagram before presenting the project as fully tested.
