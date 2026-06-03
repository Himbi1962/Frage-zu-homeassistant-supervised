### **Phase 2: Modellierung dokumentieren**

```bash
cat > docs/02_DATENMODELLIERUNG.md << 'EOF'
# Phase 2: Datenmodellierung

## 🏠 Gebäudemodell

### Raum-Profile
rooms ( room_id, name, floor, heiz_type (Fußboden/Radiator), volume_m3, isolation_level, thermal_capacity, nominal_temp, usage_schedule_json )

### Gebäude-Thermik-Modell
- **Zeitkonstante pro Raum:** τ = f(Isolation, Volumen, Heiztyp)
- **Aufheiz-Zuschlag:** Berechnet aus historischen Daten
- **Abkühl-Verhalten:** Exponentielles Modell

## 🔥 Wärmepumpen-Inverse-Kennlinie

**Maschinenkenndaten von Vitocal 250-A:**
- COP-Kurven (Viessmann-Datenblatt)
- Abhängig von: Außentemp (TA), Rücklauftemp (RLT), Vorlauftemp (VLT-Soll)
- **Inverter-Unstetigkeitsstelle:** Verschiedene Modi unterhalb/oberhalb

**Lernprozess:**
- Sammle: (TA, RLT, VLT-Soll) → (tatsächlicher_Stromverbrauch, Wärmeleistung)
- Berechne: f_cop_unter(TA, RLT, VLT) und f_cop_über(TA, RLT, VLT)
- Validiere gegen Herstellerkurven

## 💾 Speicher-Modelle

### Sailer-Pufferspeicher (1200L)
- 3-Zonen-Modell: Oben (Warmwasser 400L), Mitte, Unten (Heizung 800L)
- Schichtungsmodell (ρ, Cp)
- Wärmeverluste ~2-3% pro 24h

### Vitocharge Vx3 (15 kWh)
- Lade-/Entladekennlinie (Effizienz)
- SOC → Spannungskurve (für EMS-Regelung)

## ⚙️ Stellventil-Charakteristiken

**Monatliche Kalibrierung:**
1. Messe pro Heizkreis: Welche Ventile sind voll offen?
2. Berechne: Sollte alle ~gleich lange offen sein
3. Regel: Ventile mit längeren Öffnungszeiten reduzieren
