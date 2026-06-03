# Phase 3: Kernlogik für Predictive Energy Management

## 🎯 Übergeordneter Algorithmus (stündlich)

```python
while True:
    # 1. PROGNOSE (nächste 24h)
    heat_demand_24h = predict_heat_demand()  # basierend auf Wetter, Nutzung
    electricity_prices_24h = get_electricity_prices()  # Tibber + Prognose
    pv_forecast_24h = get_pv_forecast()  # Solstat
    
    # 2. SPEICHER-OPTIMIERUNG
    for hour in range(24):
        optimal_charge_time = calculate_optimal_charge_time(
            heat_demand_24h[hour],
            electricity_prices_24h[hour],
            pv_forecast_24h[hour],
            vx3_soc, p210_soc
        )
    
    # 3. WÄRMEPUMPEN-BETRIEBSVORGABEN
    for hour in range(24):
        optimal_vorlauf = calculate_optimal_vorlauf(
            heat_demand_24h[hour],
            outside_temp_forecast[hour],
            cop_model
        )
        # Übergabe an UVR610 via Modbus
        set_wp_vorlauf_soll(optimal_vorlauf)
    
    # 4. RAUMTEMP-REGELUNG (alle 15 Min)
    for each_room:
        error = room_soll_temp - room_ist_temp
        if fenster_offen:
            stellventil_position = 0
        else:
            stellventil_position = calculate_position(error, learning_model)
        # Übergabe an MINI-ZB2GS Relais

python-automations/
├── energy_management/
│   ├── heat_demand_predictor.py      # Wärmebedarf prognostizieren
│   ├── cop_model.py                  # WP-Kennlinien
│   ├── storage_optimizer.py          # Speicher-Ladestrategie
│   ├── room_controller.py            # Raumtemp-Regelung (PID/Lernend)
│   └── cost_optimizer.py             # Kostenoptimierung
├── database/
│   ├── schema.sql
│   └── db_manager.py
├── sensors/
│   ├── ha_integration.py             # Home Assistant Daten
│   ├── modbus_client.py              # UVR610 via Synology
│   ├── can_bus_reader.py             # Vitocharge Vx3 (Waveshare)
│   └── zigbee_reader.py              # Sonoff-Geräte
└── main.py
