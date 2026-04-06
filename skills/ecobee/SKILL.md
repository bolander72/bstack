---
name: ecobee
description: "Control an Ecobee thermostat via Home Assistant API — check temperature, set holds, change modes, view sensor data, manage schedules. Use when user asks about the thermostat, temperature, heating/cooling, HVAC, or comfort settings. Also use for 'what's the temperature', 'set it to 72', 'turn on AC', 'is the heat on'. Requires Home Assistant with Ecobee integration."
---

# ecobee — Thermostat Control via Home Assistant

Control an Ecobee smart thermostat through Home Assistant's REST API. Set temperatures, manage holds, switch modes, and monitor sensors.

## Architecture

```
Agent ──REST API──▶ Home Assistant ──Integration──▶ Ecobee Cloud ──▶ Thermostat
```

## Prerequisites

- Home Assistant instance running with Ecobee integration configured
- Long-lived access token for HA API
- Token stored securely (e.g., macOS Keychain)
- Ecobee entity ID (typically `climate.my_ecobee` or similar)

## Setup

```bash
# Store HA token in keychain
security add-generic-password -a $(whoami) -s "home-assistant-api" -w "<your-token>"

# Retrieve for API calls
TOKEN=$(security find-generic-password -s "home-assistant-api" -w)
HA_URL="http://<your-ha-ip>:8123"
ENTITY="climate.my_ecobee"
```

## Core Operations

### Check current status
```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "$HA_URL/api/states/$ENTITY" | \
  python3 -c "
import json, sys
d = json.load(sys.stdin)
a = d['attributes']
print(f'Mode: {d[\"state\"]}')
print(f'Current: {a[\"current_temperature\"]}°F')
print(f'Target: {a.get(\"temperature\", \"?\")}°F')
print(f'Humidity: {a[\"current_humidity\"]}%')
print(f'HVAC action: {a.get(\"hvac_action\", \"?\")}')
print(f'Fan: {a.get(\"fan_mode\", \"?\")}')
"
```

### Set temperature
```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entity_id\":\"$ENTITY\",\"temperature\":72}" \
  "$HA_URL/api/services/climate/set_temperature"
```

### Set temperature range (auto mode)
```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entity_id\":\"$ENTITY\",\"target_temp_high\":76,\"target_temp_low\":68}" \
  "$HA_URL/api/services/climate/set_temperature"
```

### Change HVAC mode
```bash
# Options: heat, cool, heat_cool (auto), off
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entity_id\":\"$ENTITY\",\"hvac_mode\":\"cool\"}" \
  "$HA_URL/api/services/climate/set_hvac_mode"
```

### Set fan mode
```bash
# Options: auto, on
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entity_id\":\"$ENTITY\",\"fan_mode\":\"on\"}" \
  "$HA_URL/api/services/climate/set_fan_mode"
```

### Clear hold (resume schedule)
Ecobee uses "holds" to override the programmed schedule. To resume the normal schedule:
```bash
# If your HA exposes a clear_hold button entity:
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entity_id\":\"button.my_ecobee_clear_hold\"}" \
  "$HA_URL/api/services/button/press"
```

### Check hold status
```bash
# Some HA/Ecobee setups expose a current_mode select entity
curl -s -H "Authorization: Bearer $TOKEN" \
  "$HA_URL/api/states/select.my_ecobee_current_mode" | \
  python3 -c "import json,sys; d=json.load(sys.stdin); print(f'Mode: {d[\"state\"]}')"
# "unknown" typically means a custom hold is active
```

## Sensors

Ecobee integration often exposes additional sensors:

| Entity Pattern | Description |
|----------------|-------------|
| `binary_sensor.*_motion` | Thermostat motion sensor |
| `binary_sensor.*_occupancy` | Home occupancy detection |
| `sensor.*_temperature` | Remote sensor temperatures |
| `sensor.*_humidity` | Humidity readings |

### Check motion/occupancy
```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "$HA_URL/api/states/binary_sensor.my_ecobee_occupancy" | \
  python3 -c "import json,sys; print(json.load(sys.stdin)['state'])"
```

## Important Notes

### Holds
- Setting a temperature creates a **hold** that overrides the programmed schedule
- Holds persist until manually cleared or until the next scheduled transition (depending on Ecobee settings)
- **Never clear a hold unless the user explicitly asks** — they may have set it intentionally
- Check hold status before making changes if you're unsure

### Temperature Units
- Ecobee reports in the unit configured on the thermostat (usually °F in the US)
- HA API accepts the same units

### Response Time
- Temperature changes take effect immediately on the thermostat
- Actual heating/cooling depends on HVAC equipment and current conditions
- Status updates in HA may lag 30-60 seconds behind the thermostat

## Comfort Profiles

Ecobee has built-in comfort profiles (Home, Away, Sleep). These can be activated via the Ecobee app or scheduled. The HA integration exposes the current profile in attributes.

## Tips

- Check current state before making changes — user may have a specific hold set
- For "make it cooler/warmer" requests, check current temp first and adjust by 2-3 degrees
- In `heat_cool` (auto) mode, set both high and low targets
- Fan mode "on" runs continuously; "auto" runs only with heating/cooling
- If the thermostat shows "unknown" mode, a custom hold is likely active
