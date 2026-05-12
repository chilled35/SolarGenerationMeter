# ESPHome Solar Generation Monitor

An ESPHome configuration for monitoring solar (or mains) energy generation using a Wemos D1 Mini Lite (ESP8266), reading pulse output from an energy meter's LED impulse port and exposing real-time power and cumulative energy data to Home Assistant.

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | Wemos D1 Mini Lite (ESP8266) |
| Input pin | D4 |
| Meter connection | LED impulse output (LDR or optocoupler recommended) |

### Wiring

Most modern energy meters have an LED that flashes at a fixed rate per kWh. You can detect these pulses in two ways:

**Option A — Phototransistor breakout board (recommended)**
A small phototransistor module (such as those based on the TCRT5000 or a bare NPN phototransistor like the PT333-3C) responds much faster than an LDR, making it well suited to meters with high pulse rates. Position the sensor directly over the meter's impulse LED, connect VCC to 3.3V, GND to GND, and the signal output to D4. Most ready-made modules include the necessary pull-up resistor on board.

> 💡 Avoid LDRs — their relatively slow response time can cause missed pulses, particularly on meters running at 1600 pulses/kWh or above.

**Option B — Optocoupler**
Use a PC817 or similar optocoupler connected to the meter's dedicated pulse output terminals (if available). This gives a clean, noise-free signal and is the most reliable option where the meter exposes a separate S0 pulse port.

> ⚠️ Never connect the ESP directly to mains voltage or meter terminals. Always use optical isolation.

---

## Pulse Rate Configuration

> ⚠️ **This is the most important step.** The `multiply` filter values must match your meter's pulse rate or your readings will be wrong.

The config includes two filters that convert raw pulse counts to useful units:

```yaml
filters:
  - multiply: 0.06   # Power (kW)  →  60 ÷ pulses_per_kWh
```

```yaml
filters:
  - multiply: 0.001  # Energy (kWh) →  1 ÷ pulses_per_kWh
```

The defaults assume **1000 pulses per kWh**, which is common but not universal. Check your meter's datasheet or the label on its face.

### Common Pulse Rates

| Pulses per kWh | Power multiply | Energy multiply |
|---|---|---|
| 800 | `0.075` | `0.00125` |
| **1000** | **`0.06`** *(default)* | **`0.001`** *(default)* |
| 1600 | `0.0375` | `0.000625` |
| 3200 | `0.01875` | `0.0003125` |

To calculate your own values:
- **Power (kW):** `60 ÷ pulses_per_kWh`
- **Energy (kWh):** `1 ÷ pulses_per_kWh`

---

## Setup

### 1. Prerequisites

- [ESPHome](https://esphome.io/guides/installing_esphome) installed
- Home Assistant instance running
- Wemos D1 Mini Lite flashed for the first time via USB

### 2. Configure the YAML

Copy `solar-generation.yaml` and fill in your own values where placeholders appear:

| Placeholder | Replace with |
|---|---|
| `YOUR_API_ENCRYPTION_KEY` | Generate with `esphome generate-key` |
| `YOUR_OTA_PASSWORD` | A password of your choice |
| `YOUR_WIFI_SSID` | Your WiFi network name |
| `YOUR_WIFI_PASSWORD` | Your WiFi password |
| `YOUR_AP_PASSWORD` | Fallback hotspot password |
| `192.168.x.x` | Static IP for the ESP on your network |
| `192.168.x.1` | Your router/gateway IP |

Also update the `multiply` filter values as described above.

### 3. Flash

```bash
esphome run solar-generation.yaml
```

The first flash must be via USB. Subsequent updates can be done over-the-air (OTA).

### 4. Add to Home Assistant

Once the device is online, Home Assistant should auto-discover it via the native API. Accept the integration and enter your API encryption key when prompted.

---

## Sensors Provided

| Sensor | Unit | Description |
|---|---|---|
| Power Meter | kW | Current power, updated every 10 seconds |
| Energy Meter | kWh | Cumulative energy total |
| Solar Generation WiFi Signal | dBm | ESP WiFi signal strength, updated every 60 seconds |

---

## Troubleshooting

**No pulses being detected**
- Check wiring to D4
- Verify the meter's LED is firing (visible in a dark room, or with a phone camera)
- Try increasing `update_interval` temporarily to rule out timing issues

**Power readings look wrong**
- Double-check the `multiply` values against your meter's pulses/kWh rating

**Device not appearing in Home Assistant**
- Confirm the device is on the same network/VLAN as HA
- Check the API encryption key matches in both the YAML and HA integration

**OTA updates failing**
- Verify the OTA password matches in the YAML
- Ensure the ESP's static IP hasn't been reassigned

---

## Licence

MIT — free to use and adapt. Attribution appreciated but not required.
