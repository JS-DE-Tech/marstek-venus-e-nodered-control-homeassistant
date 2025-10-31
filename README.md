# Marstek Venus E Node-RED Control

A complete **Node-RED automation flow** for the **Marstek Venus E 3.0** hybrid energy storage system.  
It enables **local control (UDP + MQTT)**, full **Home Assistant discovery**, and adds smart operation modes such as **Winter Mode** and a true **Storage Mode** with charge target and grace period.

---

## 💡 Features

### ✅ General
- Local communication via UDP (no cloud dependency)
- MQTT integration with Home Assistant discovery
- Automatic topic mapping (`stat`, `cmnd`, `discovery`)
- Works with Marstek Venus E 3.0 firmware via the public JSON-RPC UDP API

### ❄ Winter Mode
Smart logic that automatically switches between operating modes based on power flow:
- **AUTO** when PV surplus is stable → battery charges itself
- **PASSIVE** when grid import is detected & SoC ≤ 50 % → battery stops discharging
- Prevents battery drain during winter and low solar periods
- Full history logging to `marstek.log`

### 🔋 Storage Mode (Manual preservation mode)
True “battery storage” mode for long-term idle periods:
- Activated manually via Home Assistant switch
- Charges battery with **-500 W** until target SoC (default 50 %)
- Automatically enters a **grace phase** (default 5 min) holding with **-250 W**
- Keeps the inverter in `Manual` even if it tries to revert to `Passive`
- After the grace period → automatically switches to `Passive`  
  → disables HA switch and releases control back to Winter Mode
- Fully local, reliable, and autonomous

### 🧠 Smart Coordination
- Winter controller automatically pauses while Storage Mode is active
- Grace period logic continues to enforce manual hold if inverter drops out
- All transitions are logged to `marstek.log` with timestamps
- Safe recovery if Node-RED restarts mid-operation

---

## 🧰 Requirements

| Component | Description |
|------------|--------------|
| **Marstek Venus E 3.0** | Hybrid inverter / battery, UDP port 30000 |
| **Node-RED** | >= v3.x |
| **Home Assistant** | Optional, via MQTT discovery |
| **MQTT Broker** | e.g. Mosquitto, running locally |
| **Shelly 3EM (optional)** | Used by Winter Mode to detect PV surplus |

---

## 🧾 Installation

1. **Import the flow**  
   - In Node-RED → Menu ☰ → *Import* → *File / Clipboard*  
   - Paste contents of `flows/marstek_venus_e_flow.json`

2. **Configure connections**  
   - Set your MQTT broker address in the MQTT node  
   - Set the local IP of your Marstek device in the UDP node

3. **Deploy** the flow  
   Node-RED will begin discovering all sensors and switches in Home Assistant.

---

## 📁 File Overview

| File | Description |
|------|--------------|
| `flows/marstek_venus_e_flow.json` | Full Node-RED flow |
| `docs/lagerung.md` | Technical description of the Storage Mode |
| `docs/winterbetrieb.md` | Details of Winter Mode operation |
| `LICENSE` | MIT License (free to use) |

---

## 🧩 Example MQTT Topics

| Purpose | Topic | Example |
|----------|--------|----------|
| State updates | `marstek/venus_e/stat/#` | Battery SoC, mode, temperature, etc. |
| Commands | `marstek/venus_e/cmnd/mode` | `Auto`, `Manual`, `Passive` |
| Storage control | `marstek/storage/cmnd/enable` | `ON` / `OFF` |
| Winter control | `marstek/winter/cmnd/winter_mode` | `ON` / `OFF` |

---

## 🧾 Logging

All events are appended to:
