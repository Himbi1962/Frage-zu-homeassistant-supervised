# Energie-Management für Viessmann Wärmepumpe

## 🏠 Projekt-Übersicht
Kostoptimales Energiemanagement für ein Einfamilienhaus mit:
- Viessmann Wärmepumpe
- Sailer Pufferspeicher
- UVR610 (Technische Alternative Regler)
- RPi5 mit Home Assistant
- Synology 1520+ (Modbus-Slave + MySQL)
- AtomS3 Lite (Stromzähler & Wärmepumpen-Daten)

## 📋 Komponenten
- **UVR610** (TA): Wärmepumpen- und Speicherregelung
- **RPi5**: Home Assistant (aktuell), später nur noch Datenerfassung
- **Synology 1520+**: MySQL-Datenbank + Modbus-Slave
- **AtomS3 Lite #1**: Stromzählerdaten (zwei Leseköpfe)
- **AtomS3 Lite #2**: Averiy P210 Wärmepumpen-Daten
- **Sonoff-Steckdose**: Nachladung der P210

## 🎯 Ziel
Langfristig HA ablösen durch Python-Automationen für prädiktives Energiemanagement.

## 📁 Projektstruktur
- `python-automations/` - Python-Logik für Automationen
- `docs/` - Dokumentation
- `config/` - Konfigurationsdateien
- `docker/` - Docker-Container für Deployment

