# SBUS Bluetooth Bridge (ESP32)

Bridge Bluetooth gamepads (via **Bluepad32**) to **SBUS** (via **Bolder Flight Systems `sbus`**) so you can control SBUS-capable robots or receivers such as the **Unitree GO2** with modern wireless controllers.

> **Tested on:** ESP32 Wemos Lite v1 and ESPDUINO-32  
> **SBUS pin:** GPIO 16 (`Serial1`)  
> **Docs:**  
> • https://wiki.theroboverse.com/en/sbus  
> • https://github.com/mechzrobotics/Unitree_GO2_SBUS.git

---

## ✨ Features

- Pairs a Bluetooth gamepad using **Bluepad32**
- Reads sticks, D-pad, and buttons
- Converts inputs to **SBUS** channel values and transmits on `Serial1`
- Implements high-level robot actions (Unlock, Fall Recover, Stand Toggle, Lights etc.) via SBUS pulse sequences

---

## 🧩 Hardware

| Item | Notes |
|------|-------|
| Board | ESP32 Wemos Lite v1 or ESPDUINO-32 (others likely work) |
| SBUS Output | GPIO 16 (`Serial1`) |
| Logic Level | SBUS is inverted (~100 kbaud); handled by `sbus` lib |
| Power | Power ESP32 and SBUS target per vendor specs |

---

## 🛠️ Software Setup

### Requirements
- **ESP32 Arduino core** (latest)
- **Libraries**
  - [Bluepad32](https://github.com/ricardoquesada/bluepad32)
  - **Bolder Flight Systems – sbus**

### Arduino IDE Installation
1. Boards Manager → install **ESP32** core  
2. Library Manager → install **Bluepad32** and **sbus**  
3. Open `SBUS_BT_Bridge.ino`, select board + port  
4. **Upload**

> The sketch calls `BP32.forgetBluetoothKeys();` on every boot to ensure clean pairing.  
> Remove that call if you want persistent pairing.

---

## 🔗 Pairing a Controller

1. Put your gamepad in **pairing mode** (Xbox / DualShock / DualSense etc.)  
2. Power the ESP32 and open Serial Monitor @ **115200 baud**  
3. Expected output:
   ```
   CALLBACK: Controller is connected, index=0
   Controller model: <name>, VID=0x..., PID=0x...
   ```
4. Wire SBUS signal (pin 16) and GND to your receiver/robot SBUS input.

---

## 🎮 Channel Mapping

| SBUS Channel | Source | Range |
|---------------|---------|--------|
| 1 (ch[0]) | Right stick X | 432 – 1552 |
| 2 (ch[1]) | Right stick Y | 432 – 1552 |
| 3 (ch[2]) | Left stick Y | 192 – 1792 |
| 4 (ch[3]) | Left stick X | 432 – 1552 |
| 5 (ch[4]) | Mode / action | 192 / 992 / 1792 |
| 6 (ch[5]) | Action pulse | 192 / 1792 |
| 7 (ch[6]) | Action pulse | 192 / 1792 |
| 8 (ch[7]) | Toggles | 192 / 992 / 1792 |
| 9 (ch[8]) | Constant | 1690 |
| 10 (ch[9]) | Constant | 154 |

Constants used in code:
```cpp
_LOW = 192
_MID = 992
_HIGH = 1792
MODE1 = 992
MODE2 = 192
MODE3 = 1792
```

---

## 🕹️ Controls → Actions

Each action sends **three SBUS pulses** (~600 µs apart) on specific channels.

| Control | Action | SBUS Sequence |
|----------|---------|---------------|
| **START** | Unlock | CH5 = MODE1, CH7 low→high→low |
| **SELECT** | Damping mode | CH5 = MODE1, CH6 low→high→low |
| **DPAD ↑** | Fall Recover | CH5 = MODE2, CH7 low→high→low |
| **DPAD ↓** | Stand toggle (high/low) | CH5 = MODE1, CH8 MID→HIGH→LOW |
| **DPAD →** | Light toggle | CH5 = MODE3, CH8 MID→LOW→MID |
| **DPAD ←** | Continuous movement | CH5 = MODE2, CH6 low→high→low |
| **R1** | Walking gait toggle | CH5 = MODE3, CH8 MID→HIGH→MID |
| **X** | Obstacle avoidance toggle | CH5 = MODE2, CH8 MID↔LOW/HIGH |
| **Y** | Dance | CH5 = MODE3, CH7 low→high→low |
| **A** | Jump | CH5 = MODE3, CH6 low→high→low |
| Sticks | Manual motion | Continuous CH1–CH4 |

Button flags ensure one-shot activation per press.

---

## ⚙️ Code Structure

- **Callbacks:** `onConnectedController()`, `onDisconnectedController()`
- **Input processing:** `processControllers()` → `processGamepad(ctl)`
- **SBUS output:** `writeSBUS(SBUS_DATA)`
- **Optional RC readback:** `readRC()` (disabled by default)
- **Main loop:** Updates Bluepad32, writes SBUS, yields with `vTaskDelay(1)`

---

## ✅ Quick Start Checklist

- [ ] Flash the sketch to ESP32  
- [ ] Connect SBUS signal (G16) and GND  
- [ ] Open Serial Monitor @ 115200 baud  
- [ ] Pair your Bluetooth controller  
- [ ] Observe “Unlock”, “Stand Toggle” logs and verify SBUS output

---

## ⚠️ Notes & Troubleshooting

- **Flag checks:** some `else if` statements use `=` instead of `==`; fix if needed.  
- **Watchdog:** keep `vTaskDelay(1)` to yield CPU.  
- **Inversion:** SBUS is inverted; `true` parameter in `SbusTx`/`SbusRx` enables it.  
- **Safety:** Test with robot lifted or disabled motors first.

---

## 📁 Repository Layout
```
.
├── SBUS_BT_Bridge.ino
└── README.md
```

---

## 📜 License
```
SPDX-License-Identifier: MIT
```

---

## 🙌 Acknowledgements

- **Bluepad32** – ESP32 Bluetooth gamepad support  
- **Bolder Flight Systems** – `sbus` library  
- Community resources:  
  - https://wiki.theroboverse.com/en/sbus  
  - https://github.com/mechzrobotics/Unitree_GO2_SBUS.git
