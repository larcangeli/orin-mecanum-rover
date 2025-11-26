# orin-mecanum-rover
Omni-directional autonomous rover built on NVIDIA Jetson Orin Nano for multi-model computer vision tasks and simulated in Isaac Sim.

## 🤖 Hardware Architecture & System Specifications

This project utilizes a **Skid-Steering Mobile Robot** architecture powered by NVIDIA Jetson Orin Nano. Unlike traditional setups that use a micro-controller (e.g., Arduino) as a bridge, this robot implements a **direct-control architecture** where the Jetson handles both high-level perception and low-level hardware abstraction.

### 🛠 Component List

* **Compute Unit:** NVIDIA Jetson Orin Nano Developer Kit (running ROS2).
* **Storage & Comms:** 128GB NVMe SSD, Waveshare AC8265 Wireless NIC.
* **Lidar:** Slamtec RPLIDAR C1 (USB connection).
* **Actuation Interface:** ARCELI PCA9685 16-Channel 12-bit PWM Driver (I2C interface).
* **Motor Drivers:** 2x BTS7960 High-Power H-Bridges (43A Max).
* **Motors:** 4x JGB37-520 12V DC Motors with integrated Quadrature Encoders (High Torque).
* **Power Source:** Single High-Capacity Powerbank with dual outputs (DC Barrel + 12V Cigarette Lighter).
* **Chassis:** OSOYOO Metal Chassis (Skid-Steer configuration).

---

### ⚡ Power Distribution System

The power system is designed to isolate **Logic/Compute** from **Traction/Inductive Loads** while using a single battery source. This prevents voltage sags from motor inrush current (start-up spikes) from rebooting the Jetson.

#### 1. Logic & Compute Circuit (Low Current / Sensitive)
* **Source:** Powerbank DC Output (12V 15A Max).
* **Connection:** Custom 18AWG DC 5.5x2.5mm Right-Angle Cable.
* **Flow:**
    * The **Jetson Orin Nano** receives regulated 12V directly.
    * The **PCA9685** logic is powered by the Jetson's 3.3V rail.
    * The **BTS7960** logic (VCC) is powered by the Jetson's 5V rail.
    * **Sensors (Lidar/IMU):** Powered via Jetson USB/GPIO rails.

#### 2. Power & Traction Circuit (High Current / Inductive)
* **Source:** Powerbank 12V Cigarette Lighter Output.
* **Connection:** Heavy-gauge 12V cable -> Split to Driver Terminals.
* **Flow:**
    * Power goes directly to the `B+` and `B-` terminals of both **BTS7960** drivers.
    * This path is physically isolated from the Jetson's delicate electronics.

> **⚠️ Safety Note:** Soft-start algorithms (ramping velocity) must be implemented in software to prevent massive current spikes (>10A) that could trigger the powerbank's over-current protection.

---

### 🔌 Wiring & Control Logic

The robot uses a **Jetson-Centric** control loop.

#### A. Actuation (Output)
* **Protocol:** I2C (Inter-Integrated Circuit).
* **Chain:** `Jetson (I2C Bus)` → `PCA9685` → `PWM Signals` → `BTS7960` → `Motors`.
* **Mapping:**
    * **Left Side Motors:** Controlled by BTS7960 #1 (connected to PCA Channels 0-1).
    * **Right Side Motors:** Controlled by BTS7960 #2 (connected to PCA Channels 2-3).

#### B. Odometry & Feedback (Input)
* **Protocol:** GPIO Interrupts.
* **Connection:** Motor Encoders (Phase A/B) are connected **directly** to the Jetson Orin Nano GPIO header.
* **Implementation:** High-speed GPIO reading (via C++ or optimized Python `Jetson.GPIO`) captures ticks to calculate odometry and perform PID velocity control.

### 📊 System Diagram

```mermaid
graph TD
    %% Power Source
    PB[Powerbank 12V 15A]
    
    %% Main Components
    Jetson[NVIDIA Jetson Orin Nano]
    PCA[PCA9685 PWM Driver]
    DriverL[BTS7960 Driver Left]
    DriverR[BTS7960 Driver Right]
    MotorsL[Motors Left x2]
    MotorsR[Motors Right x2]
    Encoders[Encoders x4]
    Lidar[RPLIDAR C1]
    
    %% Power Lines (Red/Thick)
    PB == "12V DC Port (18AWG Cable)" ==> Jetson
    PB == "12V Cigarette Port (High Current)" ==> DriverL
    PB == "12V Cigarette Port (High Current)" ==> DriverR
    
    %% Logic Power (Blue)
    Jetson -- "3.3V Logic Power" --> PCA
    Jetson -- "5V Logic Power" --> DriverL
    Jetson -- "5V Logic Power" --> DriverR
    
    %% Data/Signals (Green/Dotted)
    Jetson -. "I2C Data" .-> PCA
    PCA -- "PWM Sig" --> DriverL
    PCA -- "PWM Sig" --> DriverR
    
    %% Motor Power
    DriverL == "Variable 12V" ==> MotorsL
    DriverR == "Variable 12V" ==> MotorsR
    
    %% Feedback Loop
    Encoders -. "GPIO Interrupts (Phase A/B)" .-> Jetson
    Lidar -. "USB Data" .-> Jetson
    
    %% Styles
    classDef power fill:#f96,stroke:#333,stroke-width:2px;
    classDef compute fill:#69f,stroke:#333,stroke-width:2px;
    classDef actuator fill:#9f6,stroke:#333,stroke-width:2px;
    
    class PB power;
    class Jetson compute;
    class DriverL,DriverR,MotorsL,MotorsR actuator;
