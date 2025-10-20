---
layout: default
title: AS5600 Magnetic Sensor
nav_order: 4
grand_parent: Archive
parent: Motor Control
---
# AS5600 Magnetic Sensor Example (SimpleFOC)

This example demonstrates how to interface the **AS5600 magnetic position sensor** using the [SimpleFOC](https://simplefoc.com/) library.  
The code initializes the sensor over the **I2C interface**, continuously reads the **angular position** and **velocity**, and prints the values to the serial monitor.

---

## 🧩 Hardware Setup

- **Sensor:** AS5600 (I2C magnetic encoder)  
- **Microcontroller:** Any Arduino-compatible board  
- **Connections:**
  - `VCC` → 5V  
  - `GND` → GND  
  - `SDA` → A4 (Arduino UNO)  
  - `SCL` → A5 (Arduino UNO)

---

## ⚙️ Code Explanation

### 1. Include Library

```cpp
#include <SimpleFOC.h>
```

The `SimpleFOC` library provides a high-level interface for magnetic sensors, BLDC control, and motor drivers.

---

### 2. Define Sensor

```cpp
// You can either use the detailed constructor or a shortcut
MagneticSensorI2C as5600 = MagneticSensorI2C(AS5600_I2C);
```

This initializes the AS5600 sensor with default parameters for I2C communication.

---

### 3. Setup Function

```cpp
void setup() {
  Serial.begin(115200);       // Initialize serial communication
  as5600.init();               // Initialize the AS5600 sensor
  Serial.println("AS5600 ready");
  _delay(1000);                // Wait a moment before starting
}
```

- Initializes communication and the AS5600 device.  
- Sends a confirmation message to the serial terminal.

---

### 4. Loop Function

```cpp
void loop() {
  as5600.update();                          // Refresh sensor data
  Serial.print(as5600.getAngle());          // Print current angle (radians)
  Serial.print("\t");
  Serial.println(as5600.getVelocity());     // Print angular velocity (rad/s)
}
```

- `getAngle()` returns the **absolute position** in radians.  
- `getVelocity()` gives the **instantaneous angular velocity**.  
- The tab `\t` separates the two values in the serial output.

---

## 🖥️ Example Output

| Angle (rad) | Velocity (rad/s) |
|--------------|------------------|
| 0.5421       | 1.2387           |
| 0.5563       | 1.2794           |
| 0.5708       | 1.3202           |

---

## 📚 References

- [SimpleFOC Documentation](https://docs.simplefoc.com/)
- [AS5600 Datasheet](https://ams.com/as5600)
