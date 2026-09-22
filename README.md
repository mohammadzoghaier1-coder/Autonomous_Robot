# 🇵🇸 Zahtar — Autonomous MicroMouse Robot

<p align="center">
  <img src="https://img.shields.io/badge/Platform-ESP32-blue" />
  <img src="https://img.shields.io/badge/Algorithm-Flood--Fill-orange" />
  <img src="https://img.shields.io/badge/Competition-2026-green" />
  <img src="https://img.shields.io/badge/Result-5th%20Place-gold" />
</p>

<p align="center">
  An autonomous maze-solving robot developed for the first MicroMouse Maze Competition in Palestine.
</p>

---

# 🏆 Competition

**Zahtar** is an autonomous MicroMouse robot developed to compete in the **first-ever MicroMouse Maze Competition in Palestine**.

The competition was established by **Code Academy** and funded by **Gaza Sky Geeks**, providing a platform for robotics and autonomous maze-solving projects in Palestine.

Our team successfully achieved:

## 🥇 5th Place

with a maze-solving time of approximately:

## ⏱️ 1 Minute

The official competition website and results can be found here:

**[🌐 MicroMouse Maze Competition](https://micromouse.site/)**

---

# 📖 Repository Contents

```text
├── floodfill.cpp
│   └── Main ESP32 Arduino firmware
│
├── micromouse.stl
│   └── 3D-printable robot chassis
│
├── Interface_Presentation.pdf
│   └── Project and design presentation
│
└── README.md
    └── Project documentation
```

---

# 🤖 Project Overview

**Zahtar** is an autonomous differential-drive MicroMouse robot designed to explore and solve a maze without human control.

The robot combines:

* 🧠 Flood-Fill maze-solving
* 📏 VL53L0X distance sensors
* 🧭 MPU6050 motion sensing
* ⚙️ Quadrature wheel encoders
* 🚗 Dual DC motors
* 📡 Bluetooth communication
* 🎛️ PID motion control
* 🧱 Front-wall detection
* 🗺️ Dynamic maze mapping

The robot senses the maze, records walls and open paths, updates its flood-fill values, selects the next direction, and moves through the maze cell by cell.

---

# 🧠 Navigation Algorithm

The main navigation algorithm used by Zahtar is **Flood-Fill**.

The implementation is divided into three phases.

## 1. First Exploration

The robot begins at the maze entrance and explores the maze while collecting information about its surroundings.

During this phase, the robot records:

* Walls
* Open paths
* Visited cells
* Traveled paths

The flood-fill values are continuously updated as new maze information is discovered.

---

## 2. Smart Exploration

During the second exploration run, the robot continues exploring the maze while giving preference to unexplored paths when multiple possible directions have the same flood value.

This allows the robot to gather additional information about the maze and improve its knowledge of the available paths.

---

## 3. Final Speed Run

Once enough information about the maze has been collected, the robot calculates a shortest confirmed path toward the center goal.

The robot then performs its final speed run.

```text
START
  │
  ▼
┌──────────────────────┐
│ First Exploration    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Maze Mapping         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Smart Exploration    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Confirmed Maze       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Shortest Path        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Final Speed Run      │
└──────────────────────┘
```

The firmware uses an **8×8 flood-fill grid** with four center cells as the goal region.

---

# 🗺️ Maze Representation

The flood-fill system maintains several data structures to represent the discovered maze.

```cpp
floodGrid
floodWalls
floodKnown
floodVisited
floodTraveled
```

| Data Structure  | Purpose                                  |
| --------------- | ---------------------------------------- |
| `floodGrid`     | Stores flood-fill distance values        |
| `floodWalls`    | Stores discovered walls                  |
| `floodKnown`    | Stores whether an edge has been explored |
| `floodVisited`  | Tracks visited cells                     |
| `floodTraveled` | Tracks paths already traveled            |

The robot maintains its current maze coordinates using:

```cpp
int floodMouseX = 0;
int floodMouseY = 0;
```

---

# ⚙️ Hardware

## ESP32

The **ESP32** is the main controller of Zahtar.

It handles:

* Motor control
* Encoder interrupts
* Distance sensors
* MPU6050 communication
* Flood-fill navigation
* PID calculations
* Wall detection
* Bluetooth communication

---

# 📏 Distance Sensors

## 2× VL53L0X

Two VL53L0X time-of-flight distance sensors are mounted on the robot to measure the distance to the left and right maze walls.

```text
                  FRONT
                    ↑

          ┌─────────────────┐
          │                 │
          │   LEFT  RIGHT   │
          │   VL53  VL53    │
          │                 │
          │     ZAHTAR      │
          │                 │
          └─────────────────┘
```

The sensors are used for:

* Side-wall detection
* Robot centering
* Lateral correction
* Wall following
* Maze mapping

The left sensor is assigned the I²C address:

```text
0x30
```

---

# 🧭 MPU6050

The MPU6050 is used to determine the robot's orientation and yaw angle.

The robot uses yaw feedback for:

* Maintaining its heading
* Correcting its orientation
* 90° turns
* 180° turns
* Navigation between maze cells

The robot uses four directional states:

```text
FORWARD  →  0°
RIGHT    →  90°
BACKWARD →  180°
LEFT     →  270°
```

These states are represented in the firmware using:

```cpp
enum LocalDirectionStates {
    FORWARD_D,
    RIGHT_D,
    BACKWARD_D,
    LEFT_D
};
```

---

# ⚙️ Motors & Encoders

Zahtar uses two independently controlled DC motors.

Each wheel is equipped with a quadrature encoder.

The encoders provide feedback for:

* Measuring distance
* Synchronizing both wheels
* Straight-line movement
* PID correction
* Detecting movement
* Controlling cell-to-cell motion

Both encoder channels are handled using ESP32 interrupts.

---

# 📐 Robot Dimensions

| Specification      |  Value |
| ------------------ | -----: |
| Chassis Diameter   | 122 mm |
| Wheel Diameter     |  46 mm |
| Cell Movement Step |  21 cm |
| Encoder Poles      |     14 |
| Motor Gear Ratio   |   29:1 |

The wheel diameter used by the firmware is:

```cpp
float wheelDiameter = 4.6;
```

where the value is measured in centimeters.

---

# 🎛️ PID Control

Multiple PID controllers are implemented to improve the robot's movement accuracy.

## Encoder PID

The encoder PID controller compares the left and right wheel movement and applies a correction to keep the robot moving straight.

```text
Left Encoder ─────┐
                  │
                  ▼
             Encoder PID
                  │
                  ▼
          Motor Correction
                  ▲
                  │
Right Encoder ────┘
```

---

## Laser PID

The side VL53L0X sensors provide continuous feedback during movement.

The robot can use:

* Both walls
* Left wall only
* Right wall only
* No side wall

to calculate a lateral correction.

This allows the robot to maintain a more centered position inside the maze.

---

## Turn PID

The MPU6050 yaw angle is used as feedback for the turning controller.

The controller calculates the difference between the target yaw and the current yaw and adjusts the motors until the robot reaches the desired heading.

---

## Distance PID

Encoder feedback is also used to control the distance traveled by the robot.

This allows Zahtar to move approximately one maze cell at a time rather than relying only on a fixed delay or motor speed.

---

# 🚗 Movement System

The main movement function is:

```cpp
MoveStraight(float targetDistance_cm)
```

Before moving, the robot performs orientation and offset corrections.

During movement it uses:

1. Encoder feedback
2. Distance PID
3. Encoder synchronization
4. Side-wall laser feedback
5. Front-wall detection
6. Motor speed correction

The motor speeds are continuously adjusted according to sensor feedback.

This allows the robot to correct its trajectory while moving through the maze.

---

# 🧱 Wall Detection

Zahtar uses both front and side sensors for maze navigation.

## Front Wall

An IR sensor is used to detect a wall in front of the robot.

```cpp
#define IR_pin 32
```

The firmware treats a LOW reading as a detected front wall.

---

## Side Walls

The left and right VL53L0X sensors are used to determine whether walls are present beside the robot.

The navigation system provides:

```cpp
WallFrontPresent()
WallLeftPresent()
WallRightPresent()
```

These readings are converted into absolute maze directions and stored in the flood-fill map.

---

# 📡 Bluetooth Communication

Zahtar uses the ESP32 Bluetooth Serial interface for debugging and monitoring.

The robot is initialized with the Bluetooth name:

```text
Zahtar
```

Bluetooth communication is used to display information such as:

* Flood-fill grid
* Maze information
* Sensor readings
* Encoder values
* Yaw angle
* Navigation status
* Debug messages

This was especially useful during development and testing because the robot could send its internal state wirelessly while operating.

---

# 🔌 Wiring & Pin Assignments

## Motor Driver

| Component       | ESP32 GPIO |
| --------------- | ---------: |
| Left Motor ENA  |    GPIO 33 |
| Left Motor IN1  |    GPIO 26 |
| Left Motor IN2  |    GPIO 25 |
| Right Motor ENA |    GPIO 12 |
| Right Motor IN1 |    GPIO 14 |
| Right Motor IN2 |    GPIO 27 |

---

## Encoders

| Encoder       | Channel | ESP32 GPIO |
| ------------- | ------- | ---------: |
| Left Encoder  | C1      |    GPIO 19 |
| Left Encoder  | C2      |    GPIO 18 |
| Right Encoder | C1      |    GPIO 16 |
| Right Encoder | C2      |    GPIO 17 |

Both channels of each encoder are connected to interrupts.

---

## VL53L0X

| Component        | ESP32 GPIO / Address |
| ---------------- | -------------------: |
| Left XSHUT       |               GPIO 5 |
| Right XSHUT      |               GPIO 4 |
| Left I²C Address |               `0x30` |

---

## Other Components

| Component       |      Connection |
| --------------- | --------------: |
| Front IR Sensor |         GPIO 32 |
| On-board LED    |          GPIO 2 |
| MPU6050         |             I²C |
| VL53L0X Sensors |             I²C |
| Bluetooth       | ESP32 Bluetooth |

---

# 🏗️ Chassis Design

The robot chassis was designed around the requirements of a compact MicroMouse robot.

The design focuses on:

* Compact dimensions
* Differential-drive movement
* Stable wheel placement
* Side sensor positioning
* Front-wall detection
* Encoder integration
* MPU6050 integration
* Easy access to electronics

The robot chassis has a diameter of approximately **122 mm** and uses **46 mm wheels**.

The repository contains the 3D-printable chassis model.

---

# 🧩 System Architecture

```text
                         ┌─────────────────────┐
                         │        ESP32        │
                         │   Main Controller   │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
       ┌────────────┐        ┌────────────┐       ┌────────────┐
       │  MPU6050   │        │  VL53L0X   │       │ IR Sensor  │
       │    Yaw     │        │  Distance  │       │   Front    │
       └─────┬──────┘        └──────┬─────┘       └──────┬─────┘
             │                      │                    │
             └──────────────────────┼────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Flood-Fill       │
                         │     Navigation      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   PID Motion        │
                         │     Control         │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                   ┌───────────┐         ┌───────────┐
                   │Left Motor │         │Right Motor│
                   └─────┬─────┘         └─────┬─────┘
                         │                     │
                         ▼                     ▼
                   Left Encoder          Right Encoder
                         │                     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                                  ESP32
```

---

# 💻 Software & Libraries

The project uses the following libraries:

```text
MPU6050_6Axis_MotionApps20
Adafruit_VL53L0X
I2Cdev
Wire
BluetoothSerial
```

The flood-fill and maze-management system also uses standard C++ containers including:

```text
vector
stack
queue
string
utility
algorithm
```

---

# 📁 Project Structure

```text
Autonomous_Robot/
│
├── floodfill.cpp
│   └── Main ESP32 Arduino firmware
│
├── micromouse.stl
│   └── 3D-printable chassis
│
├── Interface_Presentation.pdf
│   └── Project presentation
│
└── README.md
    └── Project documentation
```

---

# 🏁 Competition Achievement

## 🇵🇸 First MicroMouse Maze Competition in Palestine

Zahtar was developed and entered into the first MicroMouse Maze Competition in Palestine.

The team achieved:

```text
┌─────────────────────────────┐
│       🏆 5th PLACE          │
│                             │
│     ⏱️ ~1 MINUTE            │
│                             │
│  Maze Solving Competition   │
└─────────────────────────────┘
```

The competition was established by **Code Academy** and funded by **Gaza Sky Geeks**.

### Official Competition Website

🌐 **https://micromouse.site/**

---

# 👥 Team

This project was developed by:

### Mohammed Zogahyyer

[![GitHub](https://img.shields.io/badge/GitHub-Mohammed%20Zogahyyer-black?logo=github)](https://github.com/mohammadzoghaier1-coder)

**GitHub:**
https://github.com/mohammadzoghaier1-coder

---

### Belal Amleh

[![GitHub](https://img.shields.io/badge/GitHub-Belal--amleh-black?logo=github)](https://github.com/Belal-amleh)

**GitHub:**
https://github.com/Belal-amleh

---

### Omar Abu Fanoon

[![GitHub](https://img.shields.io/badge/GitHub-Omar%20Abu%20Fanoon-black?logo=github)](https://github.com/omarmohammadabufanoon)

**GitHub:**
https://github.com/omarmohammadabufanoon

---

# 🌟 Project Highlights

```text
🇵🇸 First MicroMouse Maze Competition in Palestine
🏆 5th Place
⏱️ ~1 Minute Maze Solving Time
🤖 Autonomous Robot
🧠 Flood-Fill Navigation
📡 ESP32
📏 VL53L0X Distance Sensors
🧭 MPU6050 Orientation Tracking
⚙️ Quadrature Encoders
🎛️ PID Motion Control
🖨️ 3D-Printed Chassis
📡 Bluetooth Debugging
```

---

# 📜 License

Add your preferred open-source license here.
