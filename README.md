# 💧 ESPHome wM-Bus Wasserzähler (Diehl IZAR RC 868)

Dieses Projekt ermöglicht das präzise Auslesen von **Diehl IZAR RC 868** Funk-Wasserzählern. Durch die Kombination eines ESP32 und eines CC1101 Funkmoduls werden Wireless M-Bus Telegramme aufgefangen, dekodiert und direkt in Home Assistant verarbeitet.

## 🚀 Highlights

* 📊 **Vollständige Dashboard-Integration**: Native Unterstützung für das Home Assistant Energie-Dashboard.
* 🧠 **Lokale Datenverarbeitung**: Verbräuche (Stunde, Tag, Woche, Jahr) werden direkt auf dem ESP32 berechnet – spart Ressourcen in HA.
* 🛡️ **Spike-Schutz**: Intelligente Logik verhindert falsche "Verbrauchs-Explosionen" nach einem Neustart.
* ⚙️ **Service-Schnittstelle**: Manuelle Korrektur der Zählerstände über HA-Dienste möglich.
* 🌡️ **Hardware-Diagnose**: Überwachung von Signalstärke (RSSI), Verbindungsqualität (LQI) und interner ESP-Temperatur.

## 🛠️ 1. Hardware-Anforderungen

### Die Komponenten
* **Mikrocontroller**: ESP32 (z. B. AZ-Delivery DevKit v4).
* **Funkmodul**: CC1101 (868 MHz Version).
  * *Blaue Platinen (E07-M1101D-TH)* werden für besseren Empfang empfohlen.
* **Antenne**: 868 MHz Stabantenne oder Lambda/4 Drahtantenne.

⚠️ **WICHTIG**: Der CC1101 darf nur mit **3.3V** betrieben werden!

## 🔌 2. Verkabelung (Wiring)

| CC1101 Pin | ESP32 GPIO | Beschreibung | Farbe (Empfehlung) | 
| ----- | ----- | ----- | ----- | 
| **VCC** | 3.3V | Spannungsversorgung | 🔴 Rot | 
| **GND** | GND | Masse | ⚫ Schwarz | 
| **MOSI** | GPIO 23 | SPI Master Out | 🟤 Braun | 
| **MISO** | GPIO 19 | SPI Master In | 🟢 Grün | 
| **SCLK** | GPIO 18 | SPI Clock | 🟣 Violett | 
| **CSN** | GPIO 5 | Chip Select (Strapping Pin*) | 🟠 Orange | 
| **GDO0** | GPIO 16 | Daten-Pin 0 | 🟡 Gelb | 
| **GDO2** | GPIO 17 | Daten-Pin 2 | ⚪ Weiß | 

> **\*Hinweis**: GPIO 5 ist ein Strapping-Pin. Sollte der ESP32 in einer Boot-Schleife hängen, nutze GPIO 14 und passe die YAML an.

## 📝 3. Installation & Konfiguration

### Schritt 1: YAML vorbereiten
Lade die `water_meter_generic.yaml` in deinen ESPHome-Ordner hoch.

### Schritt 2: Secrets anlegen
Ergänze deine `secrets.yaml`:
```yaml
wifi_ssid: "DeinWLAN"
wifi_password: "DeinPasswort"
watermeter_id: "0x23070778" # Deine 8-stellige Hex-ID
ota_password: "admin"
```

### Schritt 3: Flashen ⚡
Verbinde deinen ESP32 und flashe die Firmware. Nutze den Log-Output zur Verifizierung.

## 💡 4. Funktionsweise der Logik

### 🔄 Baseline-Initialisierung
Um Fehlmessungen zu vermeiden, nutzt die Firmware eine **Baseline-Logik**:
1. Beim ersten Start wird der erste empfangene Wert als Basis gespeichert.
2. Es wird **kein** Verbrauch berechnet (Log: `Baseline gesetzt`).
3. Erst ab dem zweiten Telegramm wird die Differenz auf die Statistiken addiert.

### 🛠️ Manuelle Korrektur
Falls die Werte abweichen, nutze in HA den Dienst:
`esphome.water_meter_esp_set_stats` mit den Variablen `h` (Stunde) und `d` (Tag).

## 📊 5. Home Assistant Energie-Dashboard
Die Sensoren sind vorkonfiguriert für LTS (Long Term Statistics):
* `device_class: water`
* `state_class: total_increasing`
* `unit_of_measurement: "m³"`

> **Hinweis**: Es kann bis zu **2 Stunden** dauern, bis HA die Statistiken für das Energie-Dashboard berechnet hat.

## ⚠️ 6. Fehlerbehebung (Troubleshooting)

| Problem | Ursache | Lösung | 
| ----- | ----- | ----- | 
| ❌ `Check connection...` | SPI-Fehler | Verkabelung prüfen (MOSI/MISO). | 
| 🔇 Keine Telegramme | Frequenz/ID | Antenne prüfen / `log_unknown: True` setzen. | 
| 🐢 Hohe CPU-Last | Debug-Logs | `log_level` auf `INFO` stellen. | 
| 🔄 Boot-Loop | Strapping Pin | CSN von GPIO 5 auf GPIO 14 legen. | 

## 🤝 Credits & Lizenz
* **Library**: [esphome-components](https://github.com/SzczepanLeon/esphome-components) by SzczepanLeon.
* **Treiber**: Wireless M-Bus Protokoll für Diehl IZAR.
* **Lizenz**: MIT

*Viel Erfolg bei deinem Smart Meter Projekt!* 💧
                
