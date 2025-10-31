## Marstek Venus E – Node-RED Flow

This flow integrates a Marstek Venus E home battery, Home Assistant (MQTT Discovery), winter/storage logic, Shelly 3EM readings, and optional InfluxDB logging.

The flow provides:
- Live telemetry (SoC, grid power, temperature, etc.)
- Automatic "Winter Mode" for smart surplus charging and battery protection
- "Storage Mode" (charge to target SoC and park)
- Watchdog for mode correction
- Home Assistant MQTT Discovery (switches, sensors, select)
- File and optional InfluxDB logging

---

## Requirements

1. **MQTT Broker**
   - Running broker (e.g. Mosquitto)
   - Default config: mqtt.local:1883 / NodeRedPublic
   - Adjust in Node `HomeAssistant-MQTT` if different or needs auth

2. **Home Assistant**
   - MQTT Discovery must be enabled
   - Entities appear automatically:
     - Winter Mode
     - Storage Mode
     - Operation Mode
     - SoC, grid power, temps, etc.

3. **Marstek Venus E**
   - Reachable on LAN
   - UDP Port: 30000
   - Set IP in Node "UDP → VenusE"
   - Example: 192.168.100.50

4. **Shelly 3EM**
   - Publishes via MQTT on:
     shellypro3em-sab7-netz/status/em:0
   - Payload must contain:
     { "total_act_power": -300 }
   - Negative = export, Positive = import

5. **Log File**
   - Path: /data/logs/marstek.log
   - Ensure writable
   - Docker example: `-v ./logs:/data/logs`

6. **InfluxDB (optional)**
   - Host: influxdb:8086
   - Database: nodered
   - Or delete the three Influx nodes if unused

---

## Setup Steps

1. Import the JSON Flow → Deploy  
2. Configure MQTT broker → Deploy  
3. Set VenusE IP → Deploy  
4. Create `/data/logs` if missing → Deploy  
5. Adjust or remove Influx nodes → Deploy  
6. Adjust Shelly topic → Deploy  
7. Click Inject "Publish MQTT Discovery (once)"  
   → Home Assistant entities appear automatically

---

## Automation Logic

- **Winter Mode** → Auto when PV surplus, Passive when grid import  
- **Storage Mode** → Manual charge to target SoC, then Passive  
- **Watchdog** → Auto-corrects stuck Passive state

---
