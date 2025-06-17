# 📸 Setup eines digitalen Bilderrahmens mit Raspberry Pi

👉 [Offizielle Anleitung (Englisch)](https://www.thedigitalpictureframe.com/how-to-build-the-best-raspberry-pi-digital-picture-frame-with-bookworm-wayland-2025-edition-pi-2-3-4-5/)

---

## 🔧 Vorbereitung des Raspberry Pi

1. Installiere das Raspberry Pi OS mit dem **Raspberry Pi Imager**.
2. Aktiviere **SSH** und verbinde dich per Terminal mit dem Pi (Benutzername: `pi`).

---

## 🔄 Systemaktualisierung

```bash
sudo apt update && sudo apt upgrade -y
```

> Führt eine Systemaktualisierung durch (Paketlisten & installierte Software).

---

## 📦 Installation notwendiger Softwarepakete

```bash
sudo apt install git libsdl2-dev xwayland labwc wlr-randr vlc -y
```

> Installiert Bibliotheken und Programme für die spätere Bildanzeige und grafische Oberfläche (Wayland + VLC).

---

## 🌍 Lokalisierung & Autologin konfigurieren

```bash
sudo raspi-config
```

Ändere folgende Optionen:

* `1 System Options` → `S5 Boot` → `B2 Console Autologin` (für Benutzer `pi`)
* `5 Localisation Options` → `L1 Locale` → wähle dein Land/Region & ggf. zusätzlich `en_US.UTF-8`

> Danach beenden und den Pi neustarten.

---

## 📁 Samba zur Dateiübertragung einrichten

```bash
sudo apt install samba -y
sudo smbpasswd -a pi
sudo nano /etc/samba/smb.conf
```

Ersetze oder ergänze `smb.conf` mit:

```ini
[global]
security = user
workgroup = WORKGROUP
server role = standalone server
map to guest = never
encrypt passwords = yes
obey pam restrictions = no
client min protocol = SMB2
client max protocol = SMB3

vfs objects = catia fruit streams_xattr
fruit:metadata = stream
fruit:model = RackMac
fruit:posix_rename = yes
fruit:veto_appledouble = no
fruit:wipe_intentionally_left_blank_rfork = yes
fruit:delete_empty_adfiles = yes

[pi]
comment = Pi Directories
browseable = yes
path = /home/pi
read only = no
create mask = 0775
directory mask = 0775
```

Samba-Dienst neu starten:

```bash
sudo /etc/init.d/smbd restart
```

---

## 🖼️ Pi3D PictureFrame installieren

```bash
mkdir venv_picframe
python -m venv /home/pi/venv_picframe
source /home/pi/venv_picframe/bin/activate
pip install picframe
```

> Installiert PictureFrame und alle Abhängigkeiten in einer virtuellen Python-Umgebung.

---

## ⚙️ PictureFrame konfigurieren

Erstelle Bildverzeichnisse und führe die Konfiguration aus:

```bash
mkdir {Pictures,DeletedPictures}
picframe -i /home/pi/
```

Drücke dreimal Enter, um die Standardwerte zu übernehmen:

```
Enter picture directory [~/Pictures]:
Enter deleted picture directory [~/DeletedPictures]:
Enter locale [en_US.UTF-8]:
```

> Die Konfigurationsdatei liegt unter `~/picframe_data/config/configuration.yaml`
> Zum Bearbeiten:

```bash
nano ~/picframe_data/config/configuration.yaml
```

---

## ▶️ Start-Skript erstellen

```bash
nano start_picframe.sh
```

Inhalt:

```bash
#!/bin/bash
source /home/pi/venv_picframe/bin/activate
picframe &
```

Speichern und ausführbar machen:

```bash
chmod +x ./start_picframe.sh
./start_picframe.sh  # zum Testen
```

---

## 🔀 Autostart beim Booten einrichten

1. Autostart-Skript:

```bash
mkdir -p ~/.config/labwc
nano ~/.config/labwc/autostart
```

Inhalt:

```bash
/home/pi/start_picframe.sh
```

2. Fensterregel (optional, für sauberes Fensterlayout):

```bash
nano ~/.config/labwc/rc.xml
```

Inhalt:

```xml
<windowRules>
    <windowRule identifier="*" serverDecoration="no" />
</windowRules>
```

3. Systemd-Dienst für den Autostart:

```bash
mkdir -p ~/.config/systemd/user/
nano ~/.config/systemd/user/picframe.service
```

Inhalt:

```ini
[Unit]
Description=PictureFrame on Pi

[Service]
ExecStart=/usr/bin/labwc
Restart=always

[Install]
WantedBy=default.target
```

Dienst aktivieren und neustarten:

```bash
systemctl --user enable picframe.service
sudo reboot
```

> **Wichtig:** Bildschirm muss angeschlossen sein, bevor du neu startest!

---

## ⏸️ PictureFrame manuell steuern

* **Starten:**

  ```bash
  systemctl --user start picframe.service
  ```

* **Stoppen:**

  ```bash
  systemctl --user stop picframe.service
  ```

* **Neustarten:**

  ```bash
  systemctl --user restart picframe.service
  ```

* **Autostart aktivieren:**

  ```bash
  systemctl --user enable picframe.service
  ```

* **Autostart deaktivieren:**

  ```bash
  systemctl --user disable picframe.service
  ```
