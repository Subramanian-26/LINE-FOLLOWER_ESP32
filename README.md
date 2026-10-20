# ESP32 PID Line Follower Robot

An ESP32-based line-following robot using 7 analog IR sensors, PID control, and a motor driver to follow a track with real-time steering correction and intersection detection.

---

## Overview

The ESP32 PID Line Follower Robot is an embedded systems project designed to autonomously follow a line using an array of seven analog sensors. The ESP32 reads sensor values, calibrates and normalizes the readings, estimates the robot's position relative to the line, and adjusts the motor speeds using a PID-based control algorithm.

The project also includes intersection detection and adjustable steering offsets to help the robot respond to track junctions.

**Key Features:**

- 7 analog IR sensors for line detection
- Automatic sensor calibration at startup
- Sensor normalization for consistent readings
- Weighted-position calculation for line tracking
- PID-based steering correction
- Independent speed control for left and right motors
- Intersection detection using edge and center sensors
- Adjustable steering offset for intersection handling
- PWM-based motor speed control
- ESP32-based real-time embedded control

---

## Components

| Component | Quantity | Purpose |
|---|---:|---|
| ESP32 Development Board | 1 | Main controller |
| Analog IR Sensors | 7 | Line detection |
| L298N-compatible Motor Driver | 1 | Motor direction and speed control |
| DC Geared Motors | 2 | Robot movement |
| Robot Chassis | 1 | Mechanical platform |
| Wheels | 2 | Locomotion |
| Battery Pack | 1 | Power supply |
| Jumper Wires | As required | Electrical connections |

*Note: The motor driver interface in the code uses separate direction pins and PWM enable pins. Verify compatibility with your actual motor driver before wiring.*

---

## Pin Connections

### IR Sensor Array

| ESP32 GPIO | Sensor | Position |
|---|---|---|
| GPIO 4 | L3 | Far left |
| GPIO 32 | L2 | Left |
| GPIO 33 | L1 | Inner left |
| GPIO 35 | CF | Center |
| GPIO 34 | R1 | Inner right |
| GPIO 39 | R2 | Right |
| GPIO 36 | R3 | Far right |

### Motor Driver

| ESP32 GPIO | Motor Driver Pin | Purpose |
|---|---|---|
| GPIO 19 | PWMA | Right motor speed (PWM) |
| GPIO 27 | AIN1 | Right motor direction |
| GPIO 26 | AIN2 | Right motor direction |
| GPIO 18 | PWMB | Left motor speed (PWM) |
| GPIO 13 | BIN1 | Left motor direction |
| GPIO 14 | BIN2 | Left motor direction |
| GPIO 23 | STBY | Motor driver standby control |

**Important:** The code assumes the driver accepts separate PWM and direction signals, with `STBY` held HIGH to enable the motors. Confirm the pin names against your particular driver board.

---

## Working Principle

The robot repeatedly reads all seven sensors and calculates its position relative to the detected line. The PID controller then determines the correction needed to steer the robot back toward the line.

### Signal Flow

```text
7 Analog IR Sensors
        ↓
Read Sensor Values
        ↓
Sensor Calibration
        ↓
Normalization (0–1000)
        ↓
Weighted Line Position
        ↓
Calculate Tracking Error
        ↓
PID Controller
        ↓
Adjust Left and Right Motor Speeds
        ↓
Robot Corrects Its Direction
        ↺
```

---

## PID Control

The controller calculates the error between the detected line position and the target position.

**1. Position Error**

```cpp
int error = pos - 3000;
```

The target position is `3000`, corresponding to the center of the seven-sensor array when the sensors are ordered from left to right and their normalized readings have the expected polarity.

**2. PID Correction**

```cpp
float PIDvalue = (Kp * P) + (Ki * I) + (Kd * D) + offset;
```

Where:

- **Kp:** Proportional gain, controlling correction based on the current error.
- **Ki:** Integral gain, accounting for accumulated error.
- **Kd:** Derivative gain, responding to changes in error.
- **P:** Scaled current error.
- **I:** Accumulated error.
- **D:** Scaled change in error.
- **offset:** Additional steering correction used by the intersection-handling logic.

**3. Motor Speed Adjustment**

```cpp
int rightSpeed = baseSpeed - pid;
int leftSpeed  = baseSpeed + pid;
```

The two motors receive different speeds to turn the robot toward the line. The correction is constrained to keep the commanded motor speeds between 0 and 255.

### Current PID Parameters

| Parameter | Value |
|---|---:|
| Base speed | 20 |
| Kp | 0.3 |
| Ki | 0.0 |
| Kd | 0.3 |
| Proportional scaling | Error / 10 |
| Derivative scaling | Change in error / 10 |

*These are the values in the current source code. They may require tuning depending on the track, sensor placement, motor characteristics, and battery voltage.*

---

## Sensor Calibration and Normalization

At startup, the ESP32 samples each sensor to estimate its minimum and maximum readings.

The raw values are then converted to a normalized range of 0–1000.

```cpp
normval[i] = map(sensorval[i],
                 minval[i],
                 maxval[i],
                 0,
                 1000);

normval[i] = constrain(normval[i], 0, 1000);
```

This helps compensate for differences in sensor readings and improves consistency during line-position estimation.

**Note:** The normalization assumes that larger readings represent stronger line detection. If your sensors produce the opposite polarity, invert the normalized values.

---

## Intersection Detection

The robot checks the two outer sensors and the central sensor region to identify a possible intersection.

```cpp
x.edge = normval[0] > 500 && normval[6] > 500;

x.center = normval[2] > 500 ||
           normval[3] > 500 ||
           normval[4] > 500;
```

An intersection is detected when both edge sensors detect the line while the defined center condition is false.

When this condition is met, the current implementation briefly commands both motors at a fixed speed and applies an adjustable steering offset based on the outer sensor readings.

*Intersection detection and steering behavior are still dependent on sensor polarity and track geometry. The current implementation is a starting point for further refinement.*

---

## Technologies Used

- **Microcontroller:** ESP32
- **Programming Language:** C++
- **Development Environment:** Arduino IDE
- **Control Algorithm:** PID control
- **Sensors:** Analog IR sensor array
- **Motor Control:** PWM and digital GPIO
- **Embedded Concepts:** ADC, calibration, sensor normalization, feedback control, and real-time motor control

---

## Future Improvements

- **PID Tuning:** Optimize the controller gains for smoother and faster tracking.
- **Improved Intersection Handling:** Add reliable junction classification and turn-selection logic.
- **Line-Loss Recovery:** Implement timed recovery and line reacquisition.
- **Adaptive Speed Control:** Reduce speed on sharp turns and difficult track sections.
- **Non-Blocking Control:** Replace blocking operations with timing-based logic for more consistent control.
- **Telemetry:** Send sensor readings, position, error, and PID values to the Serial Monitor.
- **Lap Timing:** Measure lap time and evaluate performance across repeated runs.
- **Sensor Filtering:** Reduce noise in analog readings for more stable steering.
- **Battery Monitoring:** Track battery voltage to identify power-related performance changes.

---

## Author

**Project Name:** ESP32 PID Line Follower Robot  
**Platform:** Arduino IDE  
**Microcontroller:** ESP32  
**Language:** C++  
**Version:** 1.0  
**License:** Open Source — Educational Use

