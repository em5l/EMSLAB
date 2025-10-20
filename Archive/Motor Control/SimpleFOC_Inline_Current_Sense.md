---
layout: default
title: SimpleFOC – Inline Current Sensing (2-Shunt) with AS5600 and Velocity Control
nav_order: 4
grand_parent: Archive
parent: Motor Control
---

# SimpleFOC – Inline Current Sensing (2-Shunt) with AS5600 and Velocity Control

This example demonstrates **Field-Oriented Control (FOC)** using **Inline Current Sensing (2-shunt)** on a BLDC motor with an **AS5600 I²C magnetic sensor**.  
It continuously measures **phase currents (Ia, Ib, Ic)** while performing **velocity control**.

> ⚠️ This configuration is compatible only with **SimpleFOCShield v2.0 – v2.1** hardware revisions.

---

## 🧩 Hardware Setup

- **Motor:** 14-pole-pair BLDC motor  (Check your pole pair of your motor.)
- **Driver:** SimpleFOCShield v2.x (3-PWM configuration)  
- **Sensor:** AS5600 (I²C)  
- **Current sense:** 2-shunt inline measurement  
  - Shunt resistor: 0.01 Ω  
  - Amplifier gain: 50 (V/V, e.g., INA181)  
- **Microcontroller:** Arduino-compatible board

### ⚡ Pin Connections

| Function | Pin |
|-----------|-----|
| PWM1 | 9 |
| PWM2 | 5 |
| PWM3 | 6 |
| ENABLE | 8 |
| Current sense A | A0 |
| Current sense B | A2 |
| Sensor SDA | A4 |
| Sensor SCL | A5 |
| Power | 14.8 V DC |

---

## ⚙️ Code Explanation

### 1. Include Libraries

```cpp
#include <SimpleFOC.h>
```

---

### 2. Define Motor, Driver, and Sensor

```cpp
BLDCMotor motor = BLDCMotor(14);
BLDCDriver3PWM driver = BLDCDriver3PWM(9, 5, 6, 8);
MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C);
```

- Defines the motor (14 pole pairs) and associates it with a 3-PWM driver.  
- The AS5600 sensor is used for rotor position feedback.

---

### 3. Inline Current Sense Setup

```cpp
InlineCurrentSense current_sense = InlineCurrentSense(0.01f, 50.0f, A0, A2);
```

- Two-shunt current sensing (for SimpleFOCShield v2.0 – v2.1)  
- Parameters:  
  - **Resistance:** 0.01 Ω  
  - **Gain:** 50 V/V  
  - **Pins:** A0 and A2

---

### 4. Setup Function

```cpp
void setup() {
  Serial.begin(115200);
  Wire.begin();
  Wire.setClock(400000);

  sensor.init();
  motor.linkSensor(&sensor);

  driver.voltage_power_supply = 14.8;
  driver.init();
  motor.linkDriver(&driver);

  current_sense.linkDriver(&driver);
  current_sense.init();
  motor.linkCurrentSense(&current_sense);

  motor.foc_modulation = FOCModulationType::SinePWM;
  motor.controller = MotionControlType::velocity;

  motor.init();
  motor.initFOC();

  Serial.println("FOC initialized with inline current sensing!");
}
```

- Initializes the sensor, driver, and current-sensing hardware.  
- Configures FOC modulation to **sine-PWM** and enables **velocity control**.  
- Runs full FOC alignment and calibration.

---

### 5. Main Loop

```cpp
void loop() {
  motor.loopFOC();              // FOC algorithm
  motor.move(target_speed);     // Velocity control
  PhaseCurrent_s currents = current_sense.getPhaseCurrents();

  Serial.print("Ia: "); Serial.print(currents.a, 3);
  Serial.print("\tIb: "); Serial.print(currents.b, 3);
  Serial.print("\tIc: "); Serial.println(currents.c, 3);

  delay(50);
}
```

- Executes the FOC algorithm at high frequency.  
- Moves the motor at a **target speed** (10 rad/s by default).  
- Reads and prints **phase currents (Ia, Ib, Ic)** for real-time monitoring.

---

## 🖥️ Example Serial Output

```
FOC initialized with inline current sensing!
Ia: 0.123    Ib: -0.115    Ic: -0.008
Ia: 0.120    Ib: -0.110    Ic: -0.010
Ia: 0.125    Ib: -0.118    Ic: -0.007
```

---

## 📚 References

- [SimpleFOC Documentation](https://docs.simplefoc.com/)  
- [AS5600 Datasheet – AMS](https://ams.com/as5600)  
- [SimpleFOCShield v2.0 Current Sense Guide](https://docs.simplefoc.com/simplefoc_shield_v2)  
- [INA181 Datasheet – Texas Instruments](https://www.ti.com/lit/ds/symlink/ina181.pdf)
