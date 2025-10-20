---
layout: default
title: BLDC Motor Angle Control
nav_order: 4
grand_parent: Archive
parent: Motor Control
---
# SimpleFOC – AS5600 + BLDC Motor Angle Control (Commander Interface)

This example demonstrates **angle control of a BLDC motor** using the **SimpleFOC** library with an **AS5600 I2C magnetic sensor**.  
It also includes the **Commander interface**, allowing you to send commands via the serial terminal to control the motor’s target angle in real time.

---

## 🧩 Hardware Setup

- **Motor:** BLDC (14 pole pairs, gimbal type)
- **Sensor:** AS5600 magnetic encoder (I2C)
- **Driver:** 3PWM driver (e.g., SimpleFOC Mini or custom MOSFET bridge)
- **Microcontroller:** Any Arduino-compatible board

### ⚡ Connections

| Component | Pin Connections |
|------------|----------------|
| **Driver** | PWM1 → 9, PWM2 → 5, PWM3 → 6, ENABLE → 8 |
| **Sensor** | SDA → A4, SCL → A5, VCC → 5V, GND → GND |
| **Motor**  | 3-phase connections → A, B, C terminals |
| **Power**  | 12 V DC supply (as defined in `driver.voltage_power_supply`) |

---

## ⚙️ Code Explanation

### 1. Include the SimpleFOC Library

```cpp
#include <SimpleFOC.h>
```

---

### 2. Define Instances

```cpp
BLDCMotor motor = BLDCMotor(14);                    // 14 pole pairs
BLDCDriver3PWM driver = BLDCDriver3PWM(9, 5, 6, 8); // 3-PWM driver pins
MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C); // I2C AS5600
```

---

### 3. Global Variables and Commander Setup

```cpp
float target_angle = PI / 2;        // default target angle (radians)
Commander command = Commander(Serial);  

void doTarget(char* cmd) {
  command.scalar(&target_angle, cmd);
}
```

- `Commander` lets you change control parameters via serial monitor.  
- The command function `doTarget()` links the `"T"` command to `target_angle`.

---

### 4. Setup Function

```cpp
void setup() {
  Serial.begin(115200);
  Wire.begin();

  // Initialize sensor
  sensor.init();
  motor.linkSensor(&sensor);

  // Initialize driver
  driver.voltage_power_supply = 12;
  driver.init();
  motor.linkDriver(&driver);

  // Set FOC modulation and control mode
  motor.foc_modulation = FOCModulationType::SinePWM;
  motor.controller = MotionControlType::angle;

  // Initialize motor and FOC
  motor.init();
  motor.initFOC();

  // Link Commander
  command.add('T', doTarget, "target angle [rad]");

  Serial.println("Motor ready!");
  _delay(1000);
}
```

---

### 5. Main Loop

```cpp
void loop() {
  motor.loopFOC();          // Core FOC algorithm
  motor.move(target_angle); // Move motor to target angle
  command.run();            // Listen for serial commands
}
```

---

## 🧭 Using the Commander Interface

Open the **Serial Monitor** (baud rate: `115200`) and enter commands:

| Command | Description |
|----------|--------------|
| `T0`     | Move motor to 0 rad |
| `T1.57`  | Move motor to 90° (π/2 rad) |
| `T3.14`  | Move motor to 180° (π rad) |

---

## 🖥️ Example Serial Output

```
Motor ready!
Target angle: 1.5708 rad
FOC loop running...
```

---

## 📚 References

- [SimpleFOC Documentation](https://docs.simplefoc.com/)
- [AS5600 Datasheet (AMS)](https://ams.com/as5600)
- [Commander Interface Guide](https://docs.simplefoc.com/commander)
