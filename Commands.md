# 📷 Digitaler Bilderrahmen – MQTT-Steuerung mit Home Assistant & Raspberry Pi

## 🧩 Ziel
Steuerung eines digitalen Bilderrahmens (HDMI-Monitor) per MQTT über Home Assistant – ein-/ausschalten via MQTT-Schalter.

---

## 📁 Verzeichnisstruktur

```bash
/home/pi/picframe_data/config/
│
├── hdmi-on.sh      # HDMI einschalten
├── hdmi-off.sh     # HDMI ausschalten
└── monitor_control-service  # Python-Skript mit MQTT-Steuerung
```

---

## 🔧 Shell-Skripte

### `hdmi-on.sh`
```bash
#!/bin/bash

# HDMI-Port aktivieren
wlr-randr --output HDMI-A-2 --on
```

### `hdmi-off.sh`
```bash
#!/bin/bash

# HDMI-Port aktivieren
wlr-randr --output HDMI-A-2 --off
```

> Wichtig: Skripte ausführbar machen  
```bash
chmod +x hdmi-on.sh
chmod +x hdmi-off.sh
```

---

## 🐍 Python-Skript: `monitor_control-service`

```python
#!/usr/bin/env python3

import paho.mqtt.client as mqtt
import subprocess

def hdmi_on():
    try:
        print("Turning HDMI ON...")
        subprocess.call(["/home/pi/picframe_data/config/hdmi-on.sh"])
        print("Monitor turned on successfully.")
    except Exception as e:
        print(f"An error occurred while turning HDMI on: {e}")

def hdmi_off():
    try:
        print("Turning HDMI OFF...")
        subprocess.call(["/home/pi/picframe_data/config/hdmi-off.sh"])
        print("Monitor turned off successfully.")
    except Exception as e:
        print(f"An error occurred while turning HDMI off: {e}")

def on_connect(client, userdata, flags, rc):
    print("Connected to MQTT broker with result code " + str(rc))
    if rc == 0:
        print("Connection successful. Subscribing to topic: frame/monitor/set")
        client.subscribe("frame/monitor/set", qos=0)
    else:
        print("Connection failed!")

def on_message(client, userdata, msg):
    print(f"Received message on topic {msg.topic} with payload: {msg.payload}")
    try:
        payload = msg.payload.decode()
        print(f"Decoded payload: {payload}")

        if payload == "ON":
            hdmi_on()
            client.publish("frame/monitor", "ON", qos=0, retain=True)
        elif payload == "OFF":
            hdmi_off()
            client.publish("frame/monitor", "OFF", qos=0, retain=True)
        else:
            print(f"Unknown payload received: {payload}")
    except Exception as e:
        print(f"Error processing message: {e}")

def on_subscribe(client, userdata, mid, granted_qos):
    print(f"Successfully subscribed (mid={mid}) with QoS: {granted_qos}")

client = mqtt.Client()

client.on_connect = on_connect
client.on_message = on_message
client.on_subscribe = on_subscribe

print("Connecting to MQTT broker at localhost:1883...")
client.connect("192.168.178.72", 1883, 60)

print("Entering MQTT loop...")
client.loop_forever()
```

---

## 🧠 Testbefehle (MQTT)

### Monitor EIN:
```bash
mosquitto_pub -h 192.168.178.72 -t frame/monitor/set -m "ON"
```

### Monitor AUS:
```bash
mosquitto_pub -h 192.168.178.72 -t frame/monitor/set -m "OFF"
```

### Status überprüfen:
```bash
mosquitto_sub -h 192.168.178.72 -t frame/monitor --retained-only
```

---

## 🖥️ Systemd-Service einrichten

### Datei: `/etc/systemd/system/monitor_control.service`

```ini
[Unit]
Description=Monitor Control
After=network.target

[Service]
ExecStart=/home/pi/venv_picframe/bin/python /home/pi/picframe_data/config/monitor_control-service
WorkingDirectory=/home/pi
Restart=always
User=pi
StandardOutput=journal
StandardError=journal
Environment="DISPLAY=:0"
Environment="XDG_RUNTIME_DIR=/run/user/1000"

[Install]
WantedBy=multi-user.target
```

### Aktivieren & Starten

```bash
sudo systemctl daemon-reload
sudo systemctl enable monitor_control.service
sudo systemctl start monitor_control.service
```

### Logs anzeigen

```bash
journalctl -u monitor_control.service -f
```

---

## 🏠 Home Assistant Integration (YAML)

### Anlegen Docker Image in der NAS
Image Name sollte homeassistant/home-assistant:stable sein oder latest

🚨 Wichtig
Volume muss auf `/volume1/docker/homeassistant/config:/config:rw` stehen! 
Ansonsten wird keine automatische Sicherung durchgeführt.

In den Umgebungvariablen kann noch `TZ:Europe/Berlin` eingetragen werden

### `configuration.yaml` – moderne Syntax
path: `docker/homeassistant`

```yaml
mqtt:
  switch:
    - unique_id: digital_frame_switch
      name: "Digital Picture Frame"
      icon: mdi:panorama
      state_topic: "frame/monitor"
      command_topic: "frame/monitor/set"
      payload_on: "ON"
      payload_off: "OFF"
      state_on: "ON"
      state_off: "OFF"
      qos: 0
```

Wichtig ist noch, dass das Topic `monitor/frame` im MQTT abonniert wird
Einstellungen → Geräte & Dienste → MQTT → Konfigurieren → Topic abonnieren 

### Nach Änderung:

```bash
# In Home Assistant:
Einstellungen → System → Steuerung → Konfiguration prüfen
→ Neustart
```
---

## 🧰 Nützliche Tools

```bash
# MQTT Paketinstallation
sudo apt install mosquitto mosquitto-clients

# Benutzer-ID prüfen (für XDG_RUNTIME_DIR)
id -u pi
```

---

## 📎 Hinweise

- Der `retain=True`-Flag im Script ist wichtig für die Anzeige des Status in HA.
- `vcgencmd` benötigt keinen X-Server → ideal für Headless-Setups.
- Wenn `mosquitto` nicht lokal läuft, immer IP-Adresse verwenden (`192.168.x.x`).

---

## 🧩 Erweiterungsmöglichkeiten

- Automatisches Abschalten bei Inaktivität
- Präsenz- oder Zeit-basierte Steuerung
- Integration von `binary_sensor`, z. B. für Bewegung

---
