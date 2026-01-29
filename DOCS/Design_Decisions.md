# Design Decisions

This document outlines the key design decisions made during the development of the Orin Mecanum Rover, documenting the rationale behind hardware selection, architectural changes, and software strategies.

## Hardware Choices

### Computing Architecture (Distributed)
**Decision:** Adopt a Master-Slave architecture separating high-level processing from low-level hardware control.

- **High-Level Planner**: NVIDIA Jetson Orin Nano
  - **Reasoning**:
    - Powerful AI acceleration with CUDA cores for Vision/Nav2.
    - TensorRT optimization for real-time inference.
    - Native support for ROS 2 and Isaac Sim digital twin.
    - *Constraint:* 3.3V GPIO logic and non-real-time OS kernel make it unsuitable for high-frequency encoder reading.

- **Low-Level Controller**: Arduino Nano V3 (ATmega328P)
  - **Reasoning**:
    - **Voltage Compatibility**: Operates natively at 5V, matching the JGB37-520 encoder logic levels perfectly without signal degradation.
    - **Real-Time Processing**: Handles hardware interrupts for 4 quadrature encoders without the latency issues inherent in Linux-based systems (Jetson).
    - **I/O Offloading**: Centralizes all motor PWM generation and sensor reading, freeing the Jetson CPU for AI tasks.

### Signal Integrity & Safety
- **Component**: Bi-Directional Logic Level Converter (LLC)
  - **Reasoning**: Bridging the voltage gap is critical. The Arduino transmits data at 5V, while the Jetson GPIOs are 3.3V tolerant. The LLC allows safe UART communication between the two units, preventing permanent damage to the Jetson's SoC.

### Motor Selection
- **Motor Model**: JGB37-520 (12V DC with Encoder)
  - **Reasoning**:
    - High torque-to-size ratio suitable for the rover's weight.
    - Integrated Hall-effect quadrature encoders provide high-resolution odometry data necessary for autonomous navigation.
  - **Adaptation**: The motors came with a **6.5mm D-shaft**, while standard mecanum couplings were 6mm. We utilized a **lathe** to machine the couplings to 6.5mm to ensure a mechanical fit without compromising concentricity.

### Motor Driver
- **PWM Generator**: PCA9685 16-Channel Driver
- **Power Driver**: BTS7960 (x2 Double Bridges)
  - **Reasoning**:
    - The Jetson/Arduino cannot provide enough current for the motors directly.
    - BTS7960 handles high current loads (up to 43A peak) and supports H-Bridge logic for forward/reverse control.
    - PCA9685 allows controlling all 4 motors (8 PWM signals) via just 2 I2C wires, saving valuable microcontroller pins.

### Fasteners
- **Screw Size**: M2.5
  - **Reasoning**: The JGB37-520 motor faceplate mounting holes are strictly M2.5. Standard M3 screws are too large, requiring specific sourcing to ensure secure motor mounting to the chassis.

## Software Architecture

### Communication Protocol
- **Jetson ↔ Arduino**: UART (Universal Asynchronous Receiver-Transmitter)
  - **Reasoning**: Avoids occupying USB ports on the Jetson. Provides a direct, low-latency, and vibration-resistant physical connection via GPIO headers.
- **Arduino ↔ PCA9685**: I2C (Inter-Integrated Circuit)
  - **Reasoning**: Industry standard for board-to-board control; allows daisy-chaining if future components are added.

### Framework Selection
- **Framework**: ROS 2 (Robot Operating System 2)
  - **Reasoning**:
    - Industry standard for robotics development.
    - Modular node-based architecture allows easy integration of the Arduino "node" via serial communication.
    - Built-in support for distributed systems and extensive Nav2 stack support.

### Computer Vision
- **Model**: YOLO (You Only Look Once)
- **Optimization**: TensorRT
  - **Reasoning**:
    - YOLO provides the best trade-off between speed and accuracy for object detection.
    - Compiling the model with TensorRT on the Orin Nano leverages the dedicated Tensor Cores, dramatically reducing inference latency compared to standard PyTorch execution.

## Mechanical Design

### Wheel Configuration
- **Type**: Mecanum wheels
  - **Reasoning**:
    - Omni-directional movement capability (holonomic drive).
    - Eliminates the need for a complex steering mechanism (Ackermann) or skidding (Differential drive).
    - Significantly enhanced maneuverability in tight indoor spaces.

## Simulation Environment

### Platform
- **Simulator**: NVIDIA Isaac Sim
- **Format**: USD (Universal Scene Description)
  - **Reasoning**:
    - Native integration with the Jetson platform (Hardware-in-the-Loop testing).
    - Photorealistic rendering for vision training.
    - Physics-accurate simulation for testing mecanum vectoring logic before real-world deployment.

---

*This document will be updated as the project progresses.*
