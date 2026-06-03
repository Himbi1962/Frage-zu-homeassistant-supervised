# Phase 1: Datenerfassung & -speicherung

## 🔌 Sensornetzwerk (ZigBee über HA)

### Fußbodenheizung - Smart Thermostat Integration
- **3x SNZB-02D**: Raumtemperatur & Feuchte (Heizkreise 1-3)
- **3x Tür-/Fensterkontakte**: Fenster-Status pro Raum
- **3x elektrische Stellantriebe**: Heizkreis-Regelventile
- **1x Relais**: Pumpenfreigabe (wird durch UVR610 ersetzt)

**Aktueller Zustand:** Smart Thermostat PID nicht gut eingelernt
**Ziel:** Python-Logik übernimmt Steuerung der 3 Relais

## 🔋 Stromspeicherung

### Vitocharge Vx3 (15 kWh)
- **Laderegler:** Viessmann-interne Logik (kein direkter Zugriff)
- **Mindeststrom:** 5% SOC
- **Ladelogik:** PV-Überschuss nach Hausverbrauchsabzug
- **Datenbezug:** 
  - Vx3-CAN-Bus → Waveshare 2-CAN-TO-ETH
  - Eigener ECM-Stromzähler im CAN
  - **Problem:** ECM-Werte vs. EVU-Zähler-Abweichungen
  - **Lösung:** Hausverbrauch aus (PV - Netzeinspeisung) berechnen

### Averiy P210 (2 kWh)
- **Steuerung:** AtomS3 #2 (Bluetooth)
- **Ladeverhalten:** Nur bei PV-Überschuss
- **Verbraucher:** Synology, Switches, Kühlschränke, USV
- **TODO:** Ladestrom-Register identifizieren

## 📊 Messwerte

### Alle 15 Minuten erfassen:
- **Außenklima:** Temperatur, Wind, Solarstrahlung (OpenWeather + Solstat)
- **Raumklima:** 3x Temp/Feuchte (SNZB-02D)
- **Fenster/Türen:** 3x Öffnungszustände
- **Wärmepumpe:** Vorlauf Ist/Soll, Stromverbrauch, Leistung (UVR610 → Modbus)
- **Pufferspeicher:** 3x Temperaturen (UVR610 → Modbus)
- **Elektro:** PV-Leistung, Vx3 SOC, P210 SOC, Netzleistung, Strompreis
- **Stellventile:** Öffnungspositionen (Feedback vom UVR610)

### Zielstruktur MySQL:
```sql
measurements (
  timestamp,
  -- Außen
  außentemp, windgeschwindigkeit, solarstrahlung,
  -- Räume
  raum1_ist, raum1_soll, raum1_fenster, raum1_stellventil_pos,
  raum2_ist, raum2_soll, raum2_fenster, raum2_stellventil_pos,
  raum3_ist, raum3_soll, raum3_fenster, raum3_stellventil_pos,
  -- Wärmepumpe
  wp_vorlauf_ist, wp_vorlauf_soll, wp_strom, wp_wärmeleistung,
  -- Speicher
  puffer_oben, puffer_mitte, puffer_unten,
  -- Elektro
  pv_leistung, vx3_soc, p210_soc, netz_leistung, strompreis,
  -- Pumpe
  pumpe_freigabe
