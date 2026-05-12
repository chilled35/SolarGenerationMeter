ESPHome pulse counter config for solar/mains energy monitoring on a Wemos D1 Mini Lite (ESP8266). Counts pulses from an energy meter's LED impulse output and converts them to power (kW) and cumulative energy (kWh), with results exposed to Home Assistant via the native API.

⚠️ Important: The multiply filter values (0.06 and 0.001) are calculated for a meter with 1000 pulses per kWh. If your meter uses a different pulse rate (commonly 800, 1600, or 3200 pulses/kWh), you'll need to adjust these values accordingly:

Power (kW): 60 ÷ pulses_per_kWh
Energy (kWh): 1 ÷ pulses_per_kWh
