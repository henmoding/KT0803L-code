# 📻 ESP32 + KT0803L FM Transmitter

A simple project to build an **FM transmitter** using an ESP32 (including ESP32-C3) and the KT0803L.

- Adjustable frequency
- Adjustable transmit power (0–15)
- Controlled via I2C

---

## 🛒 Where to buy

You can get the module here:

👉 https://www.ebay.de/itm/406215067142

Typical names:
- "FM Transmitter Module 87–108 MHz"
- "KT0803L Audio Transmitter Board"

---

## 🔧 Hardware

- ESP32 / ESP32-C3 Super Mini
- KT0803L FM Transmitter module
- Optional: antenna (~75 cm wire)

---

## 🔌 Wiring

### I2C Connection

| KT0803L | ESP32 |
|--------|------|
| SDA    | GPIO 8 |
| SCL    | GPIO 9 |
| VCC    | 3.3V |
| GND    | GND |

> ⚠️ Note: If it doesn't work, add pull-up resistors (4.7kΩ) to SDA and SCL.

---

## 🚀 Features

- Frequency range: 70–108 MHz  
- Adjustable transmit power  
- Stereo capable  
- Very low power consumption  

---

## 📡 Example Code

```cpp
#include <Wire.h>

#define KT0803L_ADDR 0x3E

uint16_t currentFreq = 0;

void writeRegister(uint8_t reg, uint8_t val) {
  Wire.beginTransmission(KT0803L_ADDR);
  Wire.write(reg);
  Wire.write(val);
  Wire.endTransmission();
  delay(2);
}

void setFrequency(float mhz) {
  currentFreq = (uint16_t)(mhz * 20);

  uint8_t reg0 = (currentFreq >> 1) & 0xFF;
  uint8_t reg1 = ((currentFreq >> 9) & 0x07) | 0xC0;
  uint8_t reg2 = ((currentFreq & 0x01) << 7) | 0x40;

  writeRegister(0x00, reg0);
  writeRegister(0x01, reg1);
  writeRegister(0x02, reg2);
}

void setPower(uint8_t power) {
  power &= 0x0F;

  uint8_t low  = power & 0x03;
  uint8_t mid  = (power >> 2) & 0x01;
  uint8_t high = (power >> 3) & 0x01;

  uint8_t reg1 = ((currentFreq >> 9) & 0x07) | (low << 6);
  uint8_t reg2 = ((currentFreq & 0x01) << 7) | (high << 6);

  writeRegister(0x01, reg1);
  writeRegister(0x02, reg2);
  writeRegister(0x13, mid << 7);
}

void setup() {
  Serial.begin(115200);

  Wire.begin(8, 9); // ESP32-C3 pins

  delay(200);

  setFrequency(100.0); // MHz
  setPower(1);         // 0–15

  Serial.println("FM transmitter running");
}

void loop() {
}
