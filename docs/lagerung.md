# 🔋 Storage Mode – Technical Details

The **Storage Mode** (German: *Lagerungsmodus*) provides a controlled procedure to charge the battery to a defined target SoC (default 50 %) and then hold that level safely before switching to Passive.

---

## 🧠 Logic Overview

| Phase | Description |
|--------|-------------|
| **Start (Manual –500 W)** | Triggered when the HA switch “Storage Mode” is set ON. Node-RED sends `Manual` with power = -500 W. |
| **Active Charging** | Runs until measured SoC ≥ target (default 50 %). |
| **Grace Phase (Manual –250 W)** | 5 minutes soft charging to stabilize voltage. If inverter drops to Passive, Node-RED forces it back to Manual (-250 W). |
| **Completion** | After grace time → automatically sets inverter to Passive, turns HA switch OFF, and clears all state flags. |

---

## 🧩 Internal Flow Variables

| Variable | Purpose |
|-----------|----------|
| `storage_active` | True = Storage mode running |
| `storage_target` | Target SoC % |
| `storage_grace_until` | UNIX ms timestamp of grace end |
| `storage_grace_mode250` | True while holding –250 W |

---

## 🧾 MQTT Topics

| Topic | Description | Example |
|--------|--------------|----------|
| `marstek/storage/cmnd/enable` | HA switch command | `ON` / `OFF` |
| `marstek/storage/stat/enable` | HA switch state | `ON` / `OFF` |
| `marstek/venus_e/stat/storage_info` | Informational messages | `storage:grace-hold mode=Manual t_left=180s` |

---

## ⚙️ Grace Enforcement

During the grace phase, two parallel watchdogs ensure stability:

1. **SoC Watcher**  
   - Reacts on battery SoC updates  
   - Periodically re-sends `Manual –250 W` to prevent automatic fallback  

2. **Mode Watcher**  
   - Reacts on mode changes from the inverter  
   - If inverter leaves `Manual`, it immediately re-asserts `Manual –250 W`

Both stop automatically after the grace timer expires.

---

## 🧩 Typical Log Example

