# 16-LED Bluetooth Heart-Shape Project (BK7231N / UAW025-T)

This project demonstrates how to create a 16-LED Heart-Shape flashing pattern display controlled via Bluetooth using a Tuya **BK7231N (UAW025-T)** Wi-Fi/BLE module and two **74HC595 Shift Registers** to expand GPIO pins. It also includes steps on flashing custom firmware using a **CH341A USB Mini Programmer**.

---

## 🛠️ Components Required
1. **UAW025-T (BK7231N)** Wi-Fi/BLE Module
2. **CH341A USB Programmer** (With Jumper)
3. **2x 74HC595 Shift Register ICs**
4. **16x LEDs** (Red color recommended)
5. **16x 220Ω or 330Ω Resistors**
6. **HC-05 or HC-06 Bluetooth Module**
7. 5V or 3.3V Power Supply & Jumper Wires

---

## 🔌 Hardware Wiring Guide

### 1. Flashing Mode (CH341A to UAW025-T)
> ⚠️ **CRITICAL:** Set the CH341A Jumper to **TTL Mode (Pins 2-3)** before plugging it in!

| CH341A Programmer | UAW025-T Module |
| :--- | :--- |
| 3V3 | 3V3 |
| GND | GND |
| TXD | RX1 |
| RXD | TX1 |
| *GND (Temporary)* | *CEN (For Resetting)* |

---

### 2. IC Connections (74HC595 Daisy-Chaining)
To control 16 LEDs with only 3 pins from the UAW025-T module, we daisy-chain two 74HC595 ICs.

* **UAW025-T Pins to ICs:**
  * **Pin 24 (DS / Data)** ➡️ IC 1 (Pin 14 - DS)
  * **Pin 26 (STCP / Latch)** ➡️ IC 1 (Pin 12) **AND** IC 2 (Pin 12) *[Connected in Parallel]*
  * **Pin 28 (SHCP / Clock)** ➡️ IC 1 (Pin 11) **AND** IC 2 (Pin 11) *[Connected in Parallel]*

* **Daisy-Chain Link (IC 1 to IC 2):**
  * IC 1 (Pin 9 - Q7') ➡️ IC 2 (Pin 14 - DS)

* **IC Power & Ground (Both ICs):**
  * Pin 16 (VCC) & Pin 10 (MR) ➡️ 3V3
  * Pin 8 (GND) & Pin 13 (OE) ➡️ GND

---

### 3. LED Connections
Connect the output pins of the ICs to the positive terminal (Anode) of each LED through a $220\ \Omega$ resistor. Connect all negative terminals (Cathodes) to GND.

* **IC 1 (Controls LEDs 1 to 8):** Pins 15, 1, 2, 3, 4, 5, 6, 7 (Q0 to Q7)
* **IC 2 (Controls LEDs 9 to 16):** Pins 15, 1, 2, 3, 4, 5, 6, 7 (Q0 to Q7)

---

### 4. Bluetooth Module (HC-05/HC-06 to UAW025-T)
> ⚠️ **NOTE:** Connect this **AFTER** flashing the firmware. Disconnect during flashing.

| HC-05 Bluetooth | UAW025-T Module |
| :--- | :--- |
| VCC | 3V3 / 5V |
| GND | GND |
| TXD | RX1 |
| RXD | TX1 |

---

## 💻 Firmware Source Code (`main.cpp`)

Compiled using **VS Code + PlatformIO** with the **LibreTiny** framework (`Generic BK7231N`).

```cpp
#include <Arduino.h>

const int dataPin = 24;   // DS
const int latchPin = 26;  // STCP
const int clockPin = 28;  // SHCP

char btData; 

void updateLEDs(uint16_t pattern);

void setup() {
  Serial.begin(9600); 
  pinMode(dataPin, OUTPUT);
  pinMode(latchPin, OUTPUT);
  pinMode(clockPin, OUTPUT);
}

void loop() {
  if (Serial.available() > 0) {
    btData = Serial.read(); 
  }

  if (btData == '1') {
    // Heartbeat Pattern
    updateLEDs(0xFFFF); delay(300);
    updateLEDs(0x0000); delay(150);
    updateLEDs(0xFFFF); delay(300);
    updateLEDs(0x0000); delay(500);
  } 
  else if (btData == '2') {
    // Chasing Pattern
    for (int i = 0; i < 16; i++) {
      if (Serial.available() > 0) { btData = Serial.read(); break; }
      updateLEDs(1 << i);
      delay(80);
    }
  } 
  else if (btData == '0') {
    // Turn OFF
    updateLEDs(0x0000);
  }
}

void updateLEDs(uint16_t pattern) {
  digitalWrite(latchPin, LOW);
  shiftOut(dataPin, clockPin, MSBFIRST, (pattern >> 8)); // High Byte (IC 2)
  shiftOut(dataPin, clockPin, MSBFIRST, (pattern & 0xFF)); // Low Byte (IC 1)
  digitalWrite(latchPin, HIGH);
}
