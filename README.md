# Marstek Venus E – Node-RED Flow

Dieser Flow integriert einen Marstek Venus E Heimspeicher, Home Assistant (MQTT Discovery), eine Winter-/Lagerungslogik, Shelly 3EM Messwerte und optional InfluxDB-Logging.

Der Flow stellt u. a. bereit:
- Live-Telemetrie des Speichers (SoC, Netzbezug, Temperatur, etc.)
- Automatik „Winterbetrieb“ (lädt nur bei Überschuss, schützt Akku bei Netzbezug/niedrigem SoC)
- „Lagerungsbetrieb“ (lädt gezielt auf definierte % und parkt dann passiv)
- Watchdog zur Fehlerkorrektur des Marstek-Modus
- Home Assistant MQTT Discovery (Schalter, Sensoren, Select)
- Logging nach Datei und optional nach InfluxDB

---

## ☑ Voraussetzungen

### 1. MQTT Broker
- Ein laufender MQTT Broker (z. B. Mosquitto).
- Der Flow erwartet standardmäßig:
  - Hostname: mqtt.local
  - Port: 1883
  - ClientID: NodeRedPublic
- Wenn dein Broker anders heißt/IP hat oder Authentifizierung erfordert:
  - In Node-RED den MQTT-Broker-Knoten HomeAssistant-MQTT öffnen und anpassen.
  - Benutzer/Passwort dort hinterlegen, falls nötig.

---

### 2. Home Assistant
- Home Assistant muss MQTT Discovery aktiviert haben.
- Nach der Discovery erscheinen automatisch:
  - Sensoren (SoC, Netzleistung, Temperatur, …)
  - Winterbetrieb (Schalter)
  - Lagerungsbetrieb (Schalter)
  - Betriebsmodus (Select / Dropdown)
- Du musst in Home Assistant nichts manuell anlegen.

---

### 3. Marstek Venus E
- Die Venus E muss im gleichen Netzwerk erreichbar sein.
- Der Flow spricht per UDP auf Port 30000.
- Im Node "UDP → VenusE" die IP-Adresse des eigenen Speichers eintragen.
  - Platzhalter im Flow ist aktuell: 192.168.100.50
  - Anpassen auf die echte IP deiner Venus E.

---

### 4. Shelly 3EM (Netz-/Hausanschlussleistung)
- Der Shelly muss aktiv Leistungsdaten per MQTT veröffentlichen.
- Der Flow hört standardmäßig auf: shellypro3em-sab7-netz/status/em:0
- Falls dein Topic anders lautet, ändere es im Node "Shelly: Netz Total Active Power" (MQTT In).

Das Shelly-Payload-Format muss total_act_power enthalten, z. B.:

{ "total_act_power": -300 }

- Negative Werte = Einspeisung
- Positive Werte = Netzbezug

---

### 5. Log-Datei / Pfad
Der Flow schreibt Status- und Ereignislogs in:
/data/logs/marstek.log

Stelle sicher, dass Node-RED auf diesen Pfad schreiben darf.

Docker-Beispiel:
-v ./logs:/data/logs

Alternativ: Pfad in diesen Nodes anpassen:
- marstek.log (file)
- read marstek.log (file in)
- overwrite → marstek.log (FIFO) (file)

---

### 6. InfluxDB (optional)
Für historisches Logging wird InfluxDB 1.x erwartet:

Host: influxdb
Port: 8086
Datenbank: nodered

Diese Konfiguration steckt im Influx-Knoten NodeRed DB.
Stelle sicher, dass die Datenbank nodered existiert oder passe den Knoten an.

Keine InfluxDB?
Dann kannst du die drei Knoten gefahrlos entfernen:
- VenusE stat/# (nur ausgewählte Felder) (MQTT In)
- → Influx: Feldnamen direkt aus Topics (Function)
- NodeRed DB – Energiespeicher (InfluxDB Out)

Der Rest des Flows läuft trotzdem weiter.

---

## 🚀 Einrichtung nach dem Import

1. Flow importieren  
   Node-RED → Menü → Import → JSON einfügen → Deploy.

2. MQTT-Broker prüfen  
   - Knoten HomeAssistant-MQTT öffnen.  
   - Broker-Adresse (mqtt.local) anpassen, falls nötig.  
   - Zugangsdaten setzen, falls dein Broker Auth nutzt.  
   - Deploy.

3. Venus E IP setzen  
   - Knoten UDP → VenusE öffnen.  
   - addr auf die IP deiner Venus E ändern.  
   - Deploy.

4. Log-Pfad sicherstellen  
   - Verzeichnis /data/logs/ anlegen (oder Pfad in den File-Nodes anpassen).  
   - Deploy.

5. InfluxDB anpassen oder deaktivieren  
   - Entweder Host/DB im Knoten NodeRed DB richtig setzen  
     oder die drei Influx-Knoten löschen.  
   - Deploy.

6. Shelly-Topic anpassen  
   - Knoten Shelly: Netz Total Active Power öffnen.  
   - Topic ggf. an dein Shelly-MQTT-Topic anpassen.  
   - Deploy.

7. Home Assistant Discovery auslösen  
   - In Node-RED den Inject-Button am Knoten Publish MQTT Discovery (once) klicken.  
   - Danach sollten in Home Assistant sofort alle Entitäten auftauchen:
     - Winterbetrieb (Switch)
     - Lagerungsbetrieb (Switch)
     - Betriebsmodus (Select)
     - Sensoren (SoC, ongrid_power, Temperaturen usw.)

---

## 🧠 Was die Automatik macht

### Winterbetrieb
Wenn Winterbetrieb = AN:
- Bei stabiler PV-Überschuss-Einspeisung → Batteriespeicher in Auto (lädt automatisch).
- Bei Netzbezug mit niedrigem SoC → Passive (verhindert Entladen ins Hausnetz).
- Wenn Lagerungsbetrieb aktiv ist, pausiert Winterbetrieb vollständig (Speicherschutz).

### Lagerungsbetrieb
Wenn Lagerungsbetrieb = AN:
- Speicher geht in Manual mit fixer Ladeleistung (z. B. -500 W).
- Lädt auf ein Ziel-SoC (standardmäßig 50 %).
- Danach „Grace Phase“ (Haltephase mit ca. -250 W), um den Ziel-SoC sauber zu treffen.
- Am Ende: setzt automatisch auf Passive, schaltet Lagerungsbetrieb auf AUS, und verhindert weiteres Laden/Entladen.

### Watchdog
- Erkennt einen bekannten Bug, bei dem die Venus E in Passive „hängenbleibt“, obwohl eigentlich Leistung fließt.
- Versucht dann automatisch einen kurzen Reset (Manual0 → Passive) und loggt den Vorgang.

---

# English Version

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
