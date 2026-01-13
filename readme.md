# ESP32-C3 Mini Board – Hardware Design Documentation

**Author:** Fabio Slika Stella
**Target MCU:** ESP32-C3-MINI-1-N4
**Purpose:** Compact ESP32-C3 development board with USB-C power, onboard 3.3 V regulation, boot/reset buttons, status LED, and breakout header.

---

## 1. Overview

This board is designed as a minimal, robust ESP32-C3 platform focused on:

* USB-C powered operation
* Simple, well-known LDO regulation (AMS1117-3.3)
* Correct USB-C device identification
* Stable power integrity for Wi‑Fi operation
* Clean reset/boot circuitry
* Easy access to GPIOs through a 1×11 header

The schematic is divided into the following functional blocks:

1. USB-C power input
2. 3.3 V linear regulation
3. ESP32-C3 core circuit
4. Boot and reset buttons
5. Status LED
6. GPIO breakout connector

---

## 2. USB-C Power Input

### 2.1 USB-C Receptacle (J3)

The USB-C connector is used only as a **5 V power source** and optionally for **USB data (D+/D−)**.

Key design points:

* **VBUS → 5 V net** supplies the regulator input.
* **GND and shield** are tied to board ground for EMC and ESD robustness.
* **D+ and D−** are routed directly to the ESP32-C3 USB pins, enabling native USB functionality (CDC/JTAG/DFU).

### 2.2 CC Configuration Resistors (R1, R2 – 5.1 kΩ)

Two 5.1 kΩ pull-down resistors are connected from **CC1 and CC2 to GND**.

This advertises the board as a **USB device (UFP)** and ensures that any USB-C charger or host enables VBUS. Without these resistors, many USB-C power sources would not provide 5 V.

This follows the USB-C specification for default current mode.

### 2.3 Input Decoupling Capacitor

A **10 µF bulk capacitor** is placed near VBUS to:

* Absorb hot‑plug inrush current
* Reduce cable-induced transients
* Provide local energy storage before the LDO

---

## 3. 3.3 V Regulation Stage

### 3.1 Regulator Choice – AMS1117-3.3 (U6)

The AMS1117-3.3 is a classic linear regulator chosen for:

* High availability and low cost
* Simple application circuit
* Ability to supply the peak current required by ESP32-C3 (~400 mA bursts)

Although not the most efficient, it is electrically forgiving and ideal for USB-powered boards.

### 3.2 Input and Output Capacitors (C5, C6, C1, C2)

* **Input:** 10 µF + 0.1 µF
* **Output:** 10 µF + 0.1 µF

Reasons for this combination:

* 10 µF provides bulk energy storage and stabilizes the LDO control loop.
* 0.1 µF ceramic handles high-frequency noise and fast current spikes.

This is critical for ESP32 devices, which draw fast, high di/dt current during RF transmissions.

### 3.3 Power LED (R3 + D1)

A green LED connected to 3.3 V through a 180 Ω resistor provides:

* Visual confirmation that the regulator is active
* Quick debugging aid for power faults

The resistor value balances brightness and current consumption.

---

## 4. ESP32-C3 Core Circuit

### 4.1 Module Choice – ESP32-C3-MINI-1-N4

This module integrates:

* ESP32-C3 RISC‑V MCU
* 4 MB flash
* RF matching network and antenna

Using the module instead of the bare chip:

* Simplifies RF design
* Reduces certification risk
* Minimizes layout complexity

### 4.2 Power Decoupling (C7, C8)

Placed close to the 3V3 pin:

* 10 µF for bulk decoupling
* 0.1 µF for high-frequency suppression

This ensures local current delivery and reduces ground bounce and digital noise.

### 4.3 Enable (EN) Circuit (R6 + C15)

* **R6 (10 kΩ)** pulls EN up to 3.3 V.
* **C15 (10 µF)** to GND forms a power‑on reset delay.

Purpose:

* Guarantees a clean rising edge on EN
* Prevents unstable startup
* Ensures the ESP32 only boots after 3.3 V is fully settled

This RC network increases reliability during slow USB ramps or noisy power conditions.

---

## 5. Boot and Reset Buttons

Two buttons are provided to control ESP32 startup modes.

### 5.1 Reset / Enable Button (SW2 – chip_pu)

This button pulls **EN (chip_pu)** low when pressed, forcing a hardware reset.

Supporting components:

* 10 kΩ pull‑up keeps EN high during normal operation.
* 0.1 µF capacitor filters contact bounce and short spikes.

### 5.2 Boot Button (SW1 – GPIO9)

GPIO9 is a strapping pin used by ESP32-C3 to enter download/bootloader mode.

Pressing this button while resetting the chip forces USB boot mode, enabling firmware flashing.

Again, the RC network improves noise immunity and button behavior.

---

## 6. USB Data Connection

D+ and D− from the USB-C connector are routed directly to the ESP32-C3 USB pins.

This allows:

* Native USB CDC (serial)
* USB-JTAG debugging
* DFU-style firmware loading

No external USB‑UART is required, reducing cost and component count.

---

## 7. GPIO Breakout Connector

### 7.1 1×11 Header (J1)

This header exposes:

* Multiple GPIOs
* RXD0 / TXD0
* Power and ground references

Design goals:

* Breadboard-friendly development
* External sensor and peripheral connection
* Programming and debugging access

The selection prioritizes commonly used pins while avoiding conflict with USB and boot strapping pins.

---

## 8. Power Integrity and EMC Considerations

Several design choices were made specifically for ESP32 stability:

* Multiple local decoupling capacitors
* RC delay on EN
* Short power paths from regulator to MCU
* Ground-referenced USB shield

These minimize:

* Brownouts during Wi‑Fi transmission
* Spurious resets
* USB disconnections
* EMI-related malfunctions

---

## 9. Design Philosophy Summary

This board follows a **"minimal but electrically solid"** philosophy:

* Proven components instead of experimental ones
* Conservative decoupling strategy
* USB-C implemented according to spec
* Explicit support for flashing and recovery

The result is a compact ESP32-C3 platform suitable for:

* Embedded product prototypes
* Firmware development
* USB-based tools
* IoT experiments and low-volume hardware projects

---

## 10. Possible Future Improvements

* Replace AMS1117 with a high‑efficiency buck or low‑IQ LDO
* Add ESD diodes on USB D+/D−
* Add current measurement shunt
* Expose additional GPIOs
* Add battery support and charger IC

---
