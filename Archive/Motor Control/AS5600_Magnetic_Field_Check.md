---
layout: default
title: SimpleFOC – AS5600 Magnetic Field Monitor
nav_order: 4
grand_parent: Archive
parent: Motor Control
---

# AS5600 Magnetic Field Monitor (I²C)

This example reads **raw magnetic field strength** and **status bits** directly from the **AS5600 magnetic encoder** via the I²C interface.  
It monitors magnetic field magnitude in millitesla (mT) and reports whether the field is too low, too high, or valid.

---

## 🧩 Hardware Setup

- **Sensor:** AS5600 (I2C magnetic position sensor)  
- **Microcontroller:** Arduino or compatible board

### ⚡ Connections

| AS5600 Pin | Arduino Pin | Description |
|-------------|-------------|-------------|
| VCC         | 5V          | Power supply |
| GND         | GND         | Ground |
| SDA         | A4          | I²C data |
| SCL         | A5          | I²C clock |

---

## ⚙️ Code Explanation

### 1. Include Libraries

```cpp
#include <Wire.h>
#include <SimpleFOC.h>
```

The code uses the standard **Wire library** for I²C communication and includes **SimpleFOC** for convenience in sensor initialization.

---

### 2. Sensor Definition

```cpp
MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C);
```

This creates an AS5600 sensor object using its default I²C address (`0x36`).

---

### 3. Helper Functions

Two helper functions read data from the AS5600 registers.

```cpp
uint8_t read8(uint8_t reg) { ... }
uint16_t read16(uint8_t reg) { ... }
```

- `read8()` reads a single byte (8 bits) from a register.  
- `read16()` reads two consecutive bytes (16 bits), combining them into a single value.

---

### 4. Setup Function

```cpp
void setup() {
  Serial.begin(115200);
  Wire.begin();
  Wire.setClock(400000); // Fast I2C mode (400 kHz)
  sensor.init();

  Serial.println(F("=== AS5600 Magnetic Field Monitor ==="));
  Serial.println(F("mag\tmT\tMD\tML\tMH"));
  Serial.println(F("------------------------------------"));
}
```

- Initializes the I²C bus and sensor.  
- Prints a table header for magnetic readings and status flags.  
- `MD`, `ML`, and `MH` correspond to the AS5600 status bits:  
  - **MD (bit 3):** Magnet detected  
  - **ML (bit 4):** Magnetic field too low  
  - **MH (bit 5):** Magnetic field too high

---

### 5. Loop Function

```cpp
void loop() {
  uint16_t mag = read16(0x1B);   // Raw magnetic magnitude
  uint8_t status = read8(0x0B);  // Status register

  bool MD = status & 0x08;
  bool ML = status & 0x10;
  bool MH = status & 0x20;

  float magnetic_field_mT = (float)mag / 4096.0f * 45.0f;

  Serial.print(mag);
  Serial.print("\t");
  Serial.print(magnetic_field_mT, 2);
  Serial.print("\t");
  Serial.print(MD);
  Serial.print("\t");
  Serial.print(ML);
  Serial.print("\t");
  Serial.println(MH);

  delay(100);
}
```

- Reads magnetic strength (`MAGNITUDE` register = 0x1B–0x1C).  
- Reads `STATUS` register = 0x0B for magnetic condition bits.  
- Converts the raw value into an **approximate field strength** in **millitesla (mT)**.  
- Prints formatted output to the serial monitor every 100 ms.

---

## 🖥️ Example Serial Output

```
=== AS5600 Magnetic Field Monitor ===
mag     mT      MD  ML  MH
------------------------------------
1820    20.00   1   0   0
1755    19.28   1   0   0
1922    21.10   1   0   0
```

---

## 📚 References

- [AS5600 Datasheet – AMS](https://ams.com/as5600)
- [SimpleFOC Documentation](https://docs.simplefoc.com/)
- [Arduino Wire Library Reference](https://www.arduino.cc/en/reference/wire)
