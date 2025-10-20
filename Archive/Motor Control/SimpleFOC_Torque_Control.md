---
layout: default
title: SimpleFOC – BLDC Motor Torque Control
nav_order: 4
grand_parent: Archive
parent: Motor Control
---
# SimpleFOC – AS5600 + BLDC Motor Torque Control (Commander Interface)

This example demonstrates **torque control** of a BLDC motor using the **SimpleFOC** library and an **AS5600 magnetic position sensor**.  
The **Commander interface** allows you to adjust the motor’s target torque in real time via the serial terminal.

---

## 🧩 Hardware Setup

- **Motor:** BLDC (14 pole pairs (choose pole pairs for your motor))
- **Sensor:** AS5600 (I2C magnetic encoder)
- **Driver:** 3PWM driver (e.g., SimpleFOC Mini or custom bridge)
- **Microcontroller:** Any Arduino‑compatible board

### ⚡ Connections

| Component | Pin Connections |
|------------|----------------|
| **Driver** | PWM1 → 9, PWM2 → 5, PWM3 → 6, ENABLE → 8 |
| **Sensor** | SDA → A4, SCL → A5, VCC → 5V, GND → GND |
| **Motor**  | Phases → A, B, C |
| **Power**  | 12 V DC supply |

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
BLDCDriver3PWM driver = BLDCDriver3PWM(9, 5, 6, 8); // 3‑PWM driver pins
MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C); // AS5600 sensor
```

---

### 3. Global Variables and Commander Setup

```cpp
float target_torque = 0.2;     // default torque (in voltage units)
Commander command = Commander(Serial);

void doTarget(char* cmd) {
  command.scalar(&target_torque, cmd);
}
```

- `target_torque` represents the control reference.  
  - If using **voltage mode**, this corresponds to output voltage.  
  - If using **DC current sensing**, it corresponds to current (Nm scaling if calibrated).  
- The **Commander** interface lets you change torque values from the serial terminal.

---

### 4. Setup Function

```cpp
void setup() {
  Serial.begin(115200);
  Wire.begin();

  // Initialize the magnetic sensor
  sensor.init();
  motor.linkSensor(&sensor);

  // Initialize driver
  driver.voltage_power_supply = 12;
  driver.init();
  motor.linkDriver(&driver);

  // Configure motor control
  motor.foc_modulation = FOCModulationType::SinePWM;
  motor.controller = MotionControlType::torque; // torque control mode

  // Initialize FOC
  motor.init();
  motor.initFOC();

  // Commander binding
  command.add('T', doTarget, "target torque [V or A]");

  Serial.println("Motor ready for torque control!");
  _delay(1000);
}
```

---

### 5. Loop Function

```cpp
void loop() {
  motor.loopFOC();           // Run core FOC algorithm
  motor.move(target_torque); // Apply torque setpoint
  command.run();             // Listen for serial commands
}
```

---

## 🧭 Using the Commander Interface

Open the **Serial Monitor** (baud rate = 115200) and send commands:

| Command | Description |
|----------|--------------|
| `T0`     | Stop the motor (zero torque) |
| `T0.2`   | Apply small forward torque |
| `T‑0.2`  | Apply small reverse torque |
| `T1`     | Apply stronger forward torque |

> 💡 Use small values (0.1–1.0) to start testing safely.  
> If the motor vibrates or overheats, reduce the torque immediately.

---

## 🖥️ Example Serial Output

```
Motor ready for torque control!
Target torque: 0.20
FOC loop running...
```

---

## 📚 References

- [SimpleFOC Documentation](https://docs.simplefoc.com/)
- [AS5600 Datasheet](https://ams.com/as5600)
- [Commander Interface Guide](https://docs.simplefoc.com/commander)
