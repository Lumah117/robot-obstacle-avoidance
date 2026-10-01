# Arduino Autonomous Obstacle Avoidance Robot

An autonomous mobile robot developed using **Arduino**, an ultrasonic distance sensor, servo-mounted environmental scanning and differential motor control.

The robot continuously monitors the environment ahead while driving. When an obstacle is detected within a defined safety distance, the robot stops, reverses, scans the available space to its left and right, and autonomously selects the direction with the greatest clearance before continuing.

This project represents an early practical introduction to **autonomous robotic behaviour**, combining sensing, decision-making and physical actuation in a complete embedded control loop.

---

## Project Overview

The robot implements a simple reactive obstacle-avoidance system:

```text
                 START
                   │
                   ▼
          Measure Distance Ahead
                   │
                   ▼
            Obstacle ≤ 20 cm?
              /           \
            No             Yes
            │               │
            ▼               ▼
      Drive Forward        Stop
                            │
                            ▼
                         Reverse
                            │
                            ▼
                           Stop
                            │
                            ▼
                       Look Right
                            │
                            ▼
                        Look Left
                            │
                            ▼
                  Compare Clearances
                       /        \
                      /          \
                     ▼            ▼
              Right Clearer   Left Clearer
                     │            │
                     ▼            ▼
                Turn Right     Turn Left
                     \            /
                      \          /
                       ▼        ▼
                    Continue Driving
```

The behaviour is entirely autonomous once the robot is powered and running.

---

## Technologies

- Arduino
- C/C++
- Ultrasonic ranging
- Servo control
- L298N motor driver
- DC motors
- Differential drive
- Reactive autonomous control
- Embedded systems

---

## Hardware

The project uses:

- Arduino-compatible microcontroller
- HC-SR04-style ultrasonic distance sensor
- Servo motor
- L298N motor driver
- Two DC drive motors
- Mobile robot chassis
- Battery/power supply

The ultrasonic sensor is mounted on the servo, allowing the robot to actively scan in different directions rather than relying only on a fixed forward-facing distance measurement.

---

## Repository Structure

```text
arduino-obstacle-avoidance-robot/
│
├── README.md
├── LICENSE
│
├── src/
│   └── obstacle_avoidance_robot.ino
│
└── media/
    └── ...
```

The `media` directory can contain photographs or other documentation of the physical robot.

---

# System Architecture

The project combines three basic elements of an autonomous robotic system:

```text
              ┌──────────────────┐
              │    PERCEPTION    │
              │                  │
              │ Ultrasonic Range │
              │      Sensor      │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │     DECISION     │
              │                  │
              │     Arduino      │
              │ Obstacle Logic   │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │    ACTUATION     │
              │                  │
              │ Servo + L298N +  │
              │    DC Motors     │
              └──────────────────┘
```

This forms a basic:

```text
Sense → Decide → Act
```

control architecture.

---

# Ultrasonic Distance Measurement

Obstacle detection is performed using an ultrasonic sensor and the `NewPing` Arduino library.

The sensor is configured using:

```cpp
#define trig_pin A1
#define echo_pin A2
#define MAXIMUM_DISTANCE 200
```

Distance measurements are obtained using:

```cpp
sonar.ping_cm();
```

providing an estimated distance to the nearest object in centimetres.

If no valid echo is returned, the implementation treats the distance as:

```text
250 cm
```

indicating that no nearby obstacle has been detected.

---

# Obstacle Detection

During normal operation, the robot continually checks the measured distance ahead.

The obstacle threshold is:

```text
20 cm
```

When:

```text
distance > 20 cm
```

the robot continues moving forward.

When:

```text
distance ≤ 20 cm
```

the obstacle-avoidance sequence is initiated.

```text
Distance Measurement
        │
        ▼
     ≤ 20 cm?
      /    \
    No      Yes
    │        │
    ▼        ▼
 Forward   Avoidance
 Driving   Sequence
```

---

# Obstacle Avoidance Sequence

When an obstacle is detected, the robot does not immediately turn in a predetermined direction.

Instead, it performs a short sequence:

```text
Obstacle Detected
       │
       ▼
      STOP
       │
       ▼
    REVERSE
       │
       ▼
      STOP
       │
       ▼
   SCAN RIGHT
       │
       ▼
    Measure
       │
       ▼
    SCAN LEFT
       │
       ▼
    Measure
       │
       ▼
Compare Distances
       │
       ▼
Turn Toward Greater Clearance
```

This gives the robot a simple method of selecting its own route around an obstacle.

---

# Servo-Based Environmental Scanning

The ultrasonic sensor is mounted on a servo motor.

During normal forward operation, the servo is centred at approximately:

```text
115°
```

When an obstacle is encountered, the sensor rotates to inspect both sides of the robot.

### Right Scan

```cpp
servo_motor.write(50);
```

The ultrasonic sensor measures the available distance to the right.

### Left Scan

```cpp
servo_motor.write(170);
```

The sensor then measures the available distance to the left.

After each scan, the servo returns to its forward-facing position.

Conceptually:

```text
                LEFT
                 170°
                  /
                 /
                /
               ●
              SERVO
                \
                 \
                  \
                  50°
                RIGHT

             Forward
               115°
```

This allows a single ultrasonic sensor to provide information about several directions.

---

# Direction Selection

After scanning both sides, the robot compares the measured clearances.

```cpp
if (distanceRight >= distanceLeft) {
    turnRight();
}
else {
    turnLeft();
}
```

The robot therefore attempts to turn toward the direction containing the greatest available space.

```text
       Right Distance
             │
             │
             ▼
        ┌─────────┐
        │ Compare │
        └─────────┘
             ▲
             │
             │
        Left Distance

       /             \
Right ≥ Left       Left > Right
     │                  │
     ▼                  ▼
Turn Right          Turn Left
```

---

# Motor Control

The two drive motors are controlled through an **L298N motor driver**.

Four Arduino outputs determine the direction of the left and right motors:

```cpp
const int LeftMotorForward = 6;
const int LeftMotorBackward = 7;

const int RightMotorForward = 5;
const int RightMotorBackward = 4;
```

By controlling each side independently, the robot can:

- Move forward
- Move backward
- Stop
- Turn left
- Turn right

---

## Forward Motion

Both motors are driven forward:

```text
Left Motor   → Forward
Right Motor  → Forward

        ROBOT
          ↑
          ↑
        Forward
```

---

## Reverse Motion

Both motors are driven backward:

```text
Left Motor   → Reverse
Right Motor  → Reverse

        ROBOT
          ↓
          ↓
        Reverse
```

---

## Turning Right

To turn right:

```text
Left Motor   → Forward
Right Motor  → Reverse
```

This causes the robot to rotate toward the right.

```text
       ↻
    ROBOT
```

---

## Turning Left

To turn left:

```text
Left Motor   → Reverse
Right Motor  → Forward
```

This causes the robot to rotate toward the left.

```text
       ↺
    ROBOT
```

This differential motor control allows the robot to perform relatively tight turns when avoiding obstacles.

---

# Software Structure

The Arduino program is divided into small functions responsible for individual robot behaviours:

```text
loop()
│
├── readPing()
│
├── moveForward()
│
├── moveBackward()
│
├── moveStop()
│
├── lookRight()
│
├── lookLeft()
│
├── turnRight()
│
└── turnLeft()
```

This separates the navigation logic from the individual hardware actions.

For example:

```cpp
moveForward();
```

describes the intended robot behaviour without requiring the main control loop to contain the individual motor-pin operations.

---

# Main Control Loop

The central control logic can be represented as:

```text
loop()
  │
  ├── Is obstacle close?
  │
  ├── YES
  │     │
  │     ├── Stop
  │     ├── Reverse
  │     ├── Stop
  │     ├── Look right
  │     ├── Look left
  │     └── Select turn direction
  │
  └── NO
        │
        └── Continue forward
```

After each cycle, another ultrasonic measurement is taken and the process repeats.

The robot therefore continually reacts to changes in its immediate environment.

---

# Arduino Libraries

The project uses two Arduino libraries.

## Servo

```cpp
#include <Servo.h>
```

Used to control the servo-mounted ultrasonic sensor.

## NewPing

```cpp
#include <NewPing.h>
```

Used to interface with the ultrasonic distance sensor.

The `NewPing` library must be installed before compiling the project if it is not already available in the Arduino environment.

---

# Running the Project

Open:

```text
src/obstacle_avoidance_robot.ino
```

using the Arduino IDE.

Install the required libraries, select the appropriate Arduino board and serial port, and upload the sketch.

Once powered, the robot will begin measuring the environment and navigating autonomously.

---

# Portfolio Cleanup

The repository preserves the original project architecture while applying several small corrections identified during portfolio review.

### Direction Comparison

The recovered original implementation measured:

```cpp
distanceRight = lookRight();
distanceLeft = lookLeft();
```

but the subsequent decision compared the previous forward-distance measurement against `distanceLeft`.

The portfolio version corrects this to:

```cpp
if (distanceRight >= distanceLeft)
```

so that the two measurements collected specifically for direction selection are compared.

### Unreachable Code

An unused delay following a `return` statement in `lookLeft()` was removed because code following the return could never execute.

### Constant Naming

The ultrasonic maximum-distance constant was renamed:

```text
maximum_distance
```

to:

```text
MAXIMUM_DISTANCE
```

for clearer constant naming.

These are minor corrections rather than a redesign of the original navigation architecture.

---

# Known Limitations

This is a relatively simple reactive obstacle-avoidance system.

The robot has no persistent representation of its environment and does not remember previously encountered obstacles.

Its behaviour is therefore:

```text
Current Sensor Reading
        │
        ▼
Immediate Decision
        │
        ▼
Motor Action
```

rather than:

```text
Sensors
   │
   ▼
Environment Map
   │
   ▼
Localisation
   │
   ▼
Path Planning
   │
   ▼
Navigation
```

Other limitations include:

- Blocking `delay()` calls
- Single ultrasonic range sensor
- No wheel encoders
- No odometry
- No localisation
- No environment mapping
- No planned trajectory
- Fixed obstacle threshold
- Fixed-duration turns
- No motor-speed feedback

These limitations are appropriate to the project's purpose as an early autonomous mobile-robot implementation.

---

# How I Would Approach It Today

A more advanced version could replace the blocking behaviour with a **non-blocking state machine**.

Instead of:

```text
Stop
 │
delay()
 │
Reverse
 │
delay()
 │
Scan
 │
delay()
```

the robot could maintain explicit navigation states:

```text
             ┌─────────────┐
             │   FORWARD   │
             └──────┬──────┘
                    │
             Obstacle Detected
                    │
                    ▼
             ┌─────────────┐
             │  REVERSING  │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │ SCAN_RIGHT  │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │  SCAN_LEFT  │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │   TURNING   │
             └──────┬──────┘
                    │
                    └──────────► FORWARD
```

Timing could then be managed using `millis()` rather than blocking delays.

---

# Potential Extensions

A more advanced version of the robot could include:

- Wheel encoders
- Closed-loop motor-speed control
- PID control
- Additional ultrasonic sensors
- Infrared proximity sensors
- LiDAR
- Inertial measurement unit
- Odometry
- Bluetooth or Wi-Fi telemetry
- ROS / ROS2 integration
- SLAM
- Path planning
- Camera-based perception

This would transform the system from a purely reactive obstacle-avoidance robot into a more capable autonomous mobile platform.

---

# Concepts Demonstrated

This project provided practical experience with:

- Arduino programming
- Embedded C/C++
- Autonomous robotics
- Ultrasonic distance sensing
- Servo control
- DC motor control
- L298N motor drivers
- Differential-drive behaviour
- Sensor-actuator integration
- Reactive control
- Obstacle detection
- Environmental scanning
- Autonomous decision-making
- Hardware/software integration

---

# Portfolio Context

This project represents an early stage in my progression from basic embedded programming into autonomous robotics.

```text
Embedded Programming
        │
        ▼
Sensor Interfacing
        │
        ▼
Motor Control
        │
        ▼
Sense → Decide → Act
        │
        ▼
Autonomous Obstacle Avoidance
        │
        ▼
Robotic Control Systems
        │
        ▼
Perception & Autonomous Systems
```

Although the navigation strategy is deliberately simple, the project demonstrates one of the fundamental ideas behind autonomous robotics: using sensor information to make decisions that directly influence physical behaviour.

It provided practical experience integrating **sensing, embedded software, decision logic and electromechanical actuation** into a functioning autonomous robot.
