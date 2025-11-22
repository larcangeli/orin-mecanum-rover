# Design Decisions

This document outlines the key design decisions made during the development of the Orin Mecanum Rover.

## Hardware Choices

### Motor Selection
- **Motor Model**: JGB37-520
- **Reasoning**: [To be documented]

### Fasteners
- **Screw Size**: M2.5 (not M3)
- **Reasoning**: [To be documented]

### Microcontroller
- **Platform**: NVIDIA Jetson Orin Nano
- **Reasoning**: 
  - Powerful AI acceleration with CUDA cores
  - TensorRT optimization for real-time inference
  - Native support for ROS 2
  - Isaac Sim compatibility for digital twin simulation

### Motor Driver
- **Controller**: PCA9685 16-Channel PWM Driver
- **Reasoning**: [To be documented]

## Software Architecture

### Framework Selection
- **Framework**: ROS 2 (Robot Operating System 2)
- **Reasoning**: 
  - Industry standard for robotics development
  - Modular node-based architecture
  - Extensive community support
  - Built-in support for distributed systems

### Computer Vision
- **Model**: YOLO (You Only Look Once)
- **Optimization**: TensorRT
- **Reasoning**: [To be documented]

## Mechanical Design

### Wheel Configuration
- **Type**: Mecanum wheels
- **Reasoning**: 
  - Omni-directional movement capability
  - No need for steering mechanism
  - Enhanced maneuverability in tight spaces

## Simulation Environment

### Platform
- **Simulator**: NVIDIA Isaac Sim
- **Format**: USD (Universal Scene Description)
- **Reasoning**: 
  - Native integration with Jetson platform
  - Photorealistic rendering
  - Physics-accurate simulation
  - Digital twin capability

---

*This document will be updated as the project progresses.*
