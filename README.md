# 📻 ESP32 + KT0803L FM Transmitter

Ein einfaches Projekt, um mit einem ESP32 (auch ESP32-C3) einen **FM-Sender** mit dem KT0803L zu betreiben.

- Frequenz frei einstellbar
- Sendeleistung einstellbar (0–15)
- Steuerung über I2C

---

## 🔧 Hardware

- ESP32 / ESP32-C3 Super Mini
- KT0803L FM Transmitter Modul
- optional: Antenne (~75 cm Draht)

---

## 🔌 Verkabelung

### I2C Verbindung

| KT0803L | ESP32 |
|--------|------|
| SDA    | GPIO 8 |
| SCL    | GPIO 9 |
| VCC    | 3.3V |
| GND    | GND |

> ⚠️ Hinweis: Falls es nicht funktioniert, Pull-Up Widerstände (4.7kΩ) an SDA und SCL hinzufügen.

---

## 🚀 Features

- Frequenzbereich: 70–108 MHz
- Einstellbare Sendeleistung
- Stereo-Unterstützung möglich
- Sehr geringer Stromverbrauch

---

## 📡 Beispiel-Code

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

  Wire.begin(8, 9); // ESP32-C3 Pins

  delay(200);

  setFrequency(100.0); // MHz
  setPower(1);         // 0–15

  Serial.println("FM Sender aktiv");
}

void loop() {
}
