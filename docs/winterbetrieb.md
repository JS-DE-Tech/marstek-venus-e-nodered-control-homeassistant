# ❄ Winter Mode – Technical Details

The **Winter Mode** optimizes energy behavior during periods of low solar generation.  
It decides between `AUTO` and `PASSIVE` based on PV surplus, grid power, and battery SoC.

---

## 🧠 Logic Overview

| Condition | Action | Purpose |
|------------|---------|---------|
| PV surplus ≤ –200 W for ≥ 30 s | → `AUTO` | Allow charging from solar |
| Grid import ≥ 100 W for ≥ 30 s and SoC ≤ 50 % | → `PASSIVE` | Prevent discharging into the house |
| Fast cutoff: SoC < 41 % | → `PASSIVE` within 5 s | Protect battery from deep discharge |
| Cooldown 60 s after PASSIVE | Blocks AUTO restart | Avoid oscillations |

---

## ⚙️ Parameters (defaults)

| Variable | Value | Description |
|-----------|--------|-------------|
| `SURPLUS_START_W` | –200 W | PV surplus threshold |
| `SURPLUS_FOR_MS` | 30 000 ms | Hold time before AUTO |
| `VGRID_STOP_W` | 100 W | Grid import threshold |
| `STOP_FOR_MS` | 30 000 ms | Hold time before PASSIVE |
| `STOP_FAST_MS` | 5 000 ms | Quick cutoff below 41 % SoC |
| `PASSIVE_COOLDOWN_MS` | 60 000 ms | Cooldown between mode changes |

---

## 🧩 Coordination with Storage Mode

When Storage mode is active (`storage_active=true`),  
the Winter controller **pauses completely** and shows a blue status  
`[PAUSE due to Storage]`.  
After Storage mode (including grace phase) ends, Winter mode resumes automatically.

---

## 🧾 Example Log

