# Zahtar — Autonomous MicroMouse Robot

## 🏆 Competition

Zahtar was developed to participate in the **first-ever MicroMouse Maze Competition in Palestine**, established and organized by **Code Academy** and funded by **Gaza Sky Geeks**.

The competition challenged participants to design and program autonomous maze-solving robots capable of navigating an unknown maze, discovering its layout, and reaching the goal as efficiently as possible.

Our robot **Zahtar achieved 5th place** in the competition, successfully solving the maze in approximately **1 minute**.

Competition website:
https://micromouse.site/

## 👥 Team Members

### Mohammed Zogahyyer

GitHub: https://github.com/mohammadzoghaier1-coder

### Belal Amleh

GitHub: https://github.com/Belal-amleh

### Omar Abu Fanoon

GitHub: https://github.com/omarmohammadabufanoon

---

## 📖 Project Overview

**Zahtar** is an autonomous MicroMouse-style maze-solving robot built using an **ESP32**.

The robot is designed to explore an unknown maze, detect walls using distance sensors, maintain its orientation and position using an IMU and wheel encoders, and calculate an efficient path to the maze's center using a **Flood-Fill navigation algorithm**.

The system combines:

* Autonomous maze exploration
* Flood-Fill path planning
* Wheel encoder feedback
* PID motor control
* IMU-based orientation control
* Laser distance sensors for wall detection and alignment
* Front-wall detection
* Automatic dead-end handling and backtracking
* Bluetooth communication for debugging and monitoring

---

## 📂 Repository Contents

```text
├── FloodFill/
│   └── Main Arduino Code
│
├── Chassis/
│   └── 3D Model / STL Files
│
├── Presentation/
│   └── Interface_Presentation.pdf
│
└── README.md
```

---

## 🧠 Navigation Algorithm

Zahtar uses a **Flood-Fill algorithm** to navigate the maze.

The maze is represented as a grid where every cell contains a flood value representing its distance from the goal.

As the robot discovers new walls, the maze representation is updated and the flood values are recalculated.

### Exploration Process

The navigation system is divided into several stages:

1. **First Exploration**

   * The robot starts from the maze entrance.
   * It detects walls while moving through the maze.
   * The discovered walls are stored in the maze representation.
   * Flood values are continuously updated.

2. **Second Exploration**

   * The robot uses the information collected during the first exploration.
   * It continues refining its understanding of the maze.
   * Previously discovered paths and walls help improve navigation.

3. **Speed Run**

   * After learning the maze, the robot calculates an efficient route.
   * It follows the known path at higher speed.
   * Movement and turning are controlled using encoder, IMU, and distance feedback.

### Goal

The Flood-Fill implementation uses the **four center cells of the maze as goal cells**, allowing the robot to recognize the center area regardless of which of the four cells it reaches.

---

## 🗺️ Maze Representation

The maze is represented using a grid containing information about:

* Cell coordinates
* Known walls
* Visited cells
* Flood values
* Traversed paths
* Goal cells

The robot updates this information during exploration.

The flood map allows the robot to select the neighboring cell with the lowest flood value while considering the walls it has discovered.

---

## 🔧 Hardware

The main hardware components used in Zahtar include:

| Component              | Purpose                             |
| ---------------------- | ----------------------------------- |
| ESP32                  | Main controller                     |
| VL53L0X ×2             | Left and right distance measurement |
| MPU6050                | Orientation and yaw measurement     |
| DC Motors ×2           | Robot movement                      |
| Quadrature Encoders ×2 | Wheel movement feedback             |
| IR Sensor              | Front-wall detection                |
| Motor Driver           | Motor control                       |
| Battery                | Power source                        |

---

## 📡 VL53L0X Distance Sensors

Zahtar uses **two VL53L0X Time-of-Flight distance sensors** to measure the distance to the left and right maze walls.

The sensors are used for:

* Wall detection
* Robot centering
* Lateral alignment
* Wall-following correction
* Detecting changes in the maze environment

The ESP32 communicates with the sensors through **I²C**.

Because both VL53L0X sensors initially use the same I²C address, the XSHUT pins are used during initialization to configure the sensors separately.

---

## 🧭 MPU6050

An **MPU6050 IMU** is used to track the robot's orientation.

The robot uses yaw information to:

* Maintain a straight heading
* Correct rotational drift
* Perform controlled 90° turns
* Perform 180° turns
* Verify the robot's final orientation after movement

---

## ⚙️ Motors & Encoders

Zahtar uses two DC motors with quadrature encoders.

The encoders provide feedback about wheel rotation and are used for:

* Measuring travelled distance
* Synchronizing the left and right wheels
* Detecting movement problems
* Controlling straight-line movement
* Controlling the robot's movement distance

The current configuration uses:

```text
Encoder poles: 14
Motor gear ratio: 29
Wheel diameter: 46 mm
```

Encoder feedback is processed using ESP32 interrupt routines.

---

## 📐 Robot Dimensions

The robot was designed specifically for MicroMouse-style maze navigation.

| Specification    |      Value |
| ---------------- | ---------: |
| Maze cell size   | 24 × 24 cm |
| Chassis diameter |     122 mm |
| Wheel diameter   |      46 mm |

The compact circular chassis allows Zahtar to rotate inside the maze cells while maintaining sufficient clearance from the walls.

---

## 🎛️ PID Control System

Several PID-based controllers are used to improve the robot's movement accuracy.

### Encoder PID

Used to maintain synchronization between the left and right wheels.

```text
Kp = 1.75
Ki = 0
Kd = 0.5
```

### Laser Distance PID

Used to maintain the robot's position relative to the maze walls.

```text
Kp = 1.75
Ki = 0
Kd = 0.5
```

### Turn PID

Used to control rotational movement and accurately reach the desired yaw angle.

```text
Kp = 1.75
Ki = 0
Kd = 0.5
```

### Movement PID

Used for accurate movement over a target distance.

```text
Kp = 0.4
Kd = 0.1
```

---

## 🚗 Movement System

The movement system combines multiple feedback sources instead of relying only on motor speed.

During a cell movement, the robot can use:

* Wheel encoder feedback
* MPU6050 yaw feedback
* Left and right VL53L0X measurements
* Front-wall detection

The robot initially travels at its base speed and gradually reduces its speed during the second half of a movement to improve stopping accuracy.

The speed is controlled between approximately **75% and 100% of the base speed** during the final part of the movement.

---

## 🧱 Wall Detection & Recovery

The robot continuously monitors the environment for unexpected obstacles or movement problems.

Wall detection can be combined with encoder feedback to identify situations where the robot is unable to continue moving.

When the robot detects that it has reached an unexpected wall or becomes stalled, the recovery system can:

1. Detect the problem.
2. Determine the appropriate reverse direction.
3. Move backward from the obstacle.
4. Reset the encoder reference.
5. Recalculate the robot's orientation.
6. Continue maze navigation.

The recovery system also uses the robot's current and previous directions to avoid immediately moving back toward the wall that caused the problem.

---

## 📡 Bluetooth Communication

Zahtar uses the ESP32 Bluetooth functionality for wireless debugging.

The Bluetooth device name is:

```text
Zahtar
```

Bluetooth output is used to monitor information such as:

* Current maze position
* Current direction
* Robot yaw
* Yaw error
* Left/right laser measurements
* Encoder values
* Movement information
* Flood-Fill state

This makes it easier to diagnose navigation and movement problems without connecting the robot directly to a computer.

---

## 🔌 Wiring & Pin Assignments

### Motor Driver

| Function        | ESP32 Pin |
| --------------- | --------: |
| Left Motor ENA  |   GPIO 33 |
| Left Motor IN1  |   GPIO 26 |
| Left Motor IN2  |   GPIO 25 |
| Right Motor ENA |   GPIO 12 |
| Right Motor IN1 |   GPIO 14 |
| Right Motor IN2 |   GPIO 27 |

### Encoders

| Encoder | Channel | ESP32 Pin |
| ------- | ------- | --------: |
| Left    | C1      |   GPIO 19 |
| Left    | C2      |   GPIO 18 |
| Right   | C1      |   GPIO 16 |
| Right   | C2      |   GPIO 17 |

### Sensors

| Component     | Signal    | ESP32 Pin |
| ------------- | --------- | --------: |
| Left VL53L0X  | XSHUT     |    GPIO 5 |
| Right VL53L0X | XSHUT     |    GPIO 4 |
| IR Sensor     | Signal    |   GPIO 32 |
| MPU6050       | Interrupt |   GPIO 15 |
| LED           | Signal    |    GPIO 2 |

### Communication

The sensors communicate with the ESP32 through the I²C interface.

The I²C clock is configured to:

```text
400 kHz
```

---

## 🏗️ Chassis Design

The robot uses a compact circular chassis with a diameter of approximately **122 mm**.

The chassis was designed to:

* Fit inside the maze cell dimensions
* Provide sufficient space for the ESP32 and electronics
* Support two drive motors
* Position the distance sensors toward the maze walls
* Maintain a balanced center of mass
* Allow the robot to rotate within a maze cell

The repository includes the chassis design files for reproduction and further modification.

---

## 🏛️ System Architecture

The overall system can be divided into several layers:

```text
                    ┌─────────────────────┐
                    │       ESP32         │
                    │   Main Controller   │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
    ┌───────────┐        ┌───────────┐        ┌───────────┐
    │ VL53L0X   │        │ MPU6050   │        │ Encoders  │
    │  Sensors  │        │    IMU    │        │           │
    └─────┬─────┘        └─────┬─────┘        └─────┬─────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │   Control System    │
                    │   PID Controllers   │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │   Motor Controller  │
                    └──────────┬──────────┘
                               ▼
                         ┌───────────┐
                         │  Motors   │
                         └───────────┘

                               │
                               ▼
                    ┌─────────────────────┐
                    │   Flood-Fill        │
                    │   Navigation        │
                    └─────────────────────┘
```

---

## 💻 Software & Libraries

The project is developed using the **Arduino framework for ESP32**.

Main libraries include:

```cpp
#include "MPU6050_6Axis_MotionApps20.h"
#include <Adafruit_VL53L0X.h>
#include "I2Cdev.h"
#include <Wire.h>
#include <BluetoothSerial.h>
```

The software combines sensor processing, motor control, PID controllers, and Flood-Fill navigation into a single autonomous control system.

---

## 📁 Project Structure

A simplified organization of the project is:

```text
Zahtar/
│
├── README.md
│
├── Code/
│   └── Zahtar.ino
│
├── Chassis/
│   └── *.stl
│
├── Presentation/
│   └── Interface_Presentation.pdf
│
└── Documentation/
    └── Images / Diagrams
```

---

## 🏆 Competition Achievement

### 5th Place — First Palestinian MicroMouse Maze Competition

Zahtar successfully competed in the first MicroMouse Maze Competition in Palestine and achieved:

```text
🏆 Position: 5th Place
⏱️ Maze solving time: ~1 minute
```

The result represents the outcome of the team's work across:

* Mechanical design
* Electronics
* Embedded programming
* Autonomous navigation
* Sensor integration
* PID control
* Maze-solving algorithms

Competition website:

https://micromouse.site/

---

## ⭐ Project Highlights

* 🤖 Autonomous maze-solving robot
* 🧠 Flood-Fill maze navigation
* 📡 Dual VL53L0X wall-distance sensing
* 🧭 MPU6050 orientation control
* ⚙️ Quadrature encoder feedback
* 🎛️ Multiple PID controllers
* 🔄 Automatic dead-end and wall recovery
* 📱 Bluetooth debugging
* 🏗️ Custom-designed chassis
* 🏆 5th place in the first MicroMouse Maze Competition in Palestine
* ⏱️ Approximately 1-minute maze solving time

---

## 🙏 Acknowledgements

Special thanks to:

* **Code Academy** — for establishing and organizing the competition.
* **Gaza Sky Geeks** — for funding and supporting the competition.
* The organizers and participants who contributed to creating the first MicroMouse competition environment in Palestine.

---

## 📜 License

This project is available for educational and research purposes.

Feel free to explore, modify, and build upon the project.
