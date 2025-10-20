---
layout: default
title: BLDC Motor Velocity Control
nav_order: 4
grand_parent: Archive
parent: Motor Control
---
# SimpleFOC – AS5600 + BLDC Motor Velocity Control (Commander Interface)

This example demonstrates **velocity control of a BLDC motor** using the **SimpleFOC** library and an **AS5600 I2C magnetic sensor**.  
The **Commander interface** allows you to send real-time commands from the serial terminal to modify the motor’s target speed.

---

## 🧩 Hardware Setup

- **Motor:** BLDC (14 pole pairs ((choose pole pairs for your motor)))
- **Sensor:** AS5600 magnetic encoder (I2C)
- **Driver:** 3PWM driver (e.g., SimpleFOC Mini or custom bridge)
- **Microcontroller:** Arduino-compatible board

### ⚡ Connections

| Component | Pin Connections |
|------------|----------------|
| **Driver** | PWM1 → 9, PWM2 → 5, PWM3 → 6, ENABLE → 8 |
| **Sensor** | SDA → A4, SCL → A5, VCC → 5V, GND → GND |
| **Motor**  | Phases → A, B, C |
| **Power**  | 12 V DC supply |

---

## ⚙️ Code Explanation

### 1. Include Library

```cpp
#include <SimpleFOC.h>
```

---

### 2. Define Instances

```cpp
BLDCMotor motor = BLDCMotor(14);                    // 14 pole pairs
BLDCDriver3PWM driver = BLDCDriver3PWM(9, 5, 6, 8); // 3-PWM driver pins
MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C); // AS5600 sensor
```

---

### 3. Global Variables and Commander Setup

```cpp
float target_velocity = 5.0;      // [rad/s] default speed
Commander command = Commander(Serial);

void doTarget(char* cmd) {
  command.scalar(&target_velocity, cmd);
}
```

- The `Commander` object enables runtime control via serial commands.  
- Typing `V10` in Serial Monitor sets the target velocity to **10 rad/s**.

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

  // Motor control configuration
  motor.foc_modulation = FOCModulationType::SinePWM;
  motor.controller = MotionControlType::velocity; // velocity control mode

  // Initialize motor and FOC
  motor.init();
  motor.initFOC();

  // Commander setup
  command.add('V', doTarget, "target velocity [rad/s]");

  Serial.println("Motor ready for velocity control!");
  _delay(1000);
}
```

---

### 5. Loop Function

```cpp
void loop() {
  motor.loopFOC();              // Run FOC algorithm
  motor.move(target_velocity);  // Apply velocity control
  command.run();                // Handle serial commands
}
```

---

## 🧭 Using the Commander Interface

Open **Serial Monitor** (baud rate `115200`) and send commands:

| Command | Description |
|----------|--------------|
| `V0`     | Stop the motor |
| `V5`     | Set speed to 5 rad/s |
| `V-5`    | Reverse rotation at 5 rad/s |

---

## 🖥️ Example Serial Output

```
Motor ready for velocity control!
Target velocity: 5.00 rad/s
FOC loop active...
```

---

## 📚 References

- [SimpleFOC Documentation](https://docs.simplefoc.com/)
- [AS5600 Datasheet](https://ams.com/as5600)
- [Commander Interface Guide](https://docs.simplefoc.com/commander)
