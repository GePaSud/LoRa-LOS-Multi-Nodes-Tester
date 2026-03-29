# LoRa Multi-Node Line-of-Sight Tester

**Hāmani Fenua · TERRA FORMA**  
Firmware for **ESP32 / Heltec WiFi LoRa 32 V2** and compatible LoRa boards

![Platform](https://img.shields.io/badge/platform-ESP32-blue)
![Radio](https://img.shields.io/badge/radio-LoRa-green)
![Interface](https://img.shields.io/badge/interface-web%20UI-orange)
![Storage](https://img.shields.io/badge/storage-LittleFS-lightgrey)
![License](https://img.shields.io/badge/license-custom-informational)

## Overview

This project turns an **ESP32 + LoRa** board into a field-ready **multi-node radio visibility tester**.

Each node:

- creates its own local **Wi-Fi access point**
- serves a **web interface** on `192.168.4.1`
- transmits LoRa beacons
- listens for packets sent by other nodes
- computes useful radio indicators
- stores received packets in **CSV logs** in **LittleFS**

The system is intended for **line-of-sight testing**, **deployment preparation**, **site comparison**, and **field validation of LoRa links**.

## Main features

- Local Wi-Fi AP with embedded web dashboard
- Node configuration from a browser
- Manual or periodic LoRa beacon transmission
- Reception and tracking of neighbouring nodes
- Radio metrics:
  - RSSI
  - SNR
  - frequency error
  - missed packets
  - duplicates
- Session-based CSV logging in LittleFS
- Optional OLED display support
- Compatible with Heltec WiFi LoRa 32 V2 and other ESP32 + LoRa boards

## Typical use case

This firmware is useful when several portable LoRa nodes are placed in the field in order to answer questions such as:

- Do these two positions see each other correctly in LoRa?
- Which location gives the most stable link?
- How does the link quality change with terrain geometry?
- What spreading factor is necessary for this route?

A smartphone can connect directly to a node’s Wi-Fi access point and display the web dashboard for live inspection.

## Web interface

After boot, each node creates a Wi-Fi AP such as:

`LoRaTest-XXXX`

Open in a browser:

`http://192.168.4.1`

The interface provides:

- **Node header** with current node identity
- **Detected nodes table**
- **Radio details**
- **Quick configuration panel**
- **Session and log export**
- **Live dashboard refresh** when valid packets are received

## Radio operating rules

All nodes in a given session must share the **exact same radio profile**:

- frequency
- spreading factor
- bandwidth
- coding rate
- TX power
- preamble length
- sync word
- CRC setting

### Important anti-collision rule

In **SF12**, use at least **10 seconds of transmission interval per node**.

Recommended minimum interval per node:

| Number of nodes | Recommended minimum interval in SF12 |
|---|---:|
| 1 | 10 s |
| 2 | 20 s |
| 3 | 30 s |
| 4 | 40 s |

**Example:**  
With **3 nodes**, configure about **30 seconds between two transmissions on each node**.

With lower spreading factors such as **SF7** or **SF9**, shorter intervals can be used.

## Supported hardware

### Fully supported
- **Heltec WiFi LoRa 32 V2**

### Supported with pin adaptation
- Other **ESP32 + LoRa** boards
- Boards **with or without display**

To port the firmware to another board, adapt:

- SPI pins
- LoRa chip select
- LoRa reset
- DIO0 / interrupt pin
- I2C pins if using an OLED

## Installation

### 1. Development environment

Install one of the following:

- **Arduino IDE 2.x**
- **PlatformIO**

Then install the **ESP32 board support package by Espressif**.

### 2. Libraries

#### Install manually

- **LoRa** by **Sandeep Mistry**
- **U8g2** only if OLED support is enabled

#### Already provided by the ESP32 core

No separate installation is normally required for:

- `WiFi.h`
- `WebServer.h`
- `DNSServer.h`
- `Preferences.h`
- `LittleFS.h`
- `SPI.h`
- `Wire.h`

### 3. Uploading the firmware

#### Arduino IDE workflow

1. Open the sketch
2. Select the board  
   For **Heltec WiFi LoRa 32 V2**, `ESP32 Dev Module` generally works well if the pins are correctly defined in code
3. Select the correct serial port
4. Compile
5. Upload

Use a **data-capable USB cable**, not a charge-only cable.

### 4. Uploading logos / favicon / static files

If you want to display properly the logos and icon, upload the `data/` folder using **LittleFS Filesystem Uploader**.

Aduino Project structure:

```text
project/
  your_sketch.ino
  data/
    logo_haamani.png
    logo_labo.png
    logo_univ.png
    logo_terraforma.png
```
## Prebuilt firmware

A prebuilt firmware package for **Heltec WiFi LoRa 32 V2** is also available in the project releases as a `.zip` archive.

This package contains:

- `sketch_mar26a_LoRa_Tester_V1.ino.bootloader.bin`
- `sketch_mar26a_LoRa_Tester_V1.ino.partitions.bin`
- `sketch_mar26a_LoRa_Tester_V1.ino.bin`

This allows users to flash the firmware **without compiling the project from source**.

### Flashing the prebuilt binaries

The easiest method is to use **esptool**.

Example command on Windows:

```bash
python -m esptool --chip esp32 --port COMX --baud 460800 write_flash -z ^
  0x1000 sketch_mar26a_LoRa_Tester_V1.ino.bootloader.bin ^
  0x8000 sketch_mar26a_LoRa_Tester_V1.ino.partitions.bin ^
  0x10000 sketch_mar26a_LoRa_Tester_V1.ino.bin
```
