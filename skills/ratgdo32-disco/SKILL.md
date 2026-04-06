---
name: ratgdo32-disco
description: Control a ratgdo32 disco garage door opener via its local web API. Use when the user asks to open/close the garage, check garage status, toggle the garage light, check if a car is parked, enable/disable remotes, or anything involving the garage door. Supports door control, light, obstruction detection, vehicle presence (laser sensor), parking assist, motion, and remote lockout. No authentication required — all local network.
---

# ratgdo32 disco — Garage Door Controller

Control a ratgdo32 disco (HomeKit firmware) garage door opener via its local REST API.

## Setup

**Device-specific values are stored in `config.json` in this skill directory.** Update `config.json` if your device IP or identity changes.

The config file contains:
- `ip` — Device IP address (used in all curl commands below)
- `mdns` — mDNS hostname
- `mac` — Device MAC address
- `homekit_name` — HomeKit accessory name

## Device Info

| Field | Value |
|-------|-------|
| **Model** | ratgdo32 disco |
| **Firmware** | HomeKit v3.4.4 |
| **Protocol** | LiftMaster Security+ 2.0 |
| **IP** | See `config.json` |
| **mDNS** | See `config.json` |
| **MAC** | See `config.json` |
| **HomeKit Name** | See `config.json` |
| **Web UI** | http://`<ip from config.json>`/ |
| **Auth** | None required (local network only) |

## Quick Reference

**Note:** All curl commands use the IP address from `config.json`.

| Action | Command |
|--------|---------|
| Get full status | `curl -s http://<ip>/status.json` |
| Open door | `curl -s -X POST -F "garageDoorState=1" http://<ip>/setgdo` |
| Close door | `curl -s -X POST -F "garageDoorState=0" http://<ip>/setgdo` |
| Light on | `curl -s -X POST -F "garageLightOn=1" http://<ip>/setgdo` |
| Light off | `curl -s -X POST -F "garageLightOn=0" http://<ip>/setgdo` |
| Disable remotes | `curl -s -X POST -F "garageLockState=1" http://<ip>/setgdo` |
| Enable remotes | `curl -s -X POST -F "garageLockState=0" http://<ip>/setgdo` |

## Status API

`GET http://<ip>/status.json` returns JSON:

```json
{
  "garageDoorState": "open|closed|opening|closing|stopped",
  "garageLightOn": true|false,
  "garageObstructed": true|false,
  "garageLockState": "locked|unlocked",
  "vehicleState": "present|absent|arriving|departing",
  "vehicleDistance": 42,
  "motionDetected": true|false
}
```

### Key fields
- **garageDoorState** — current door position
- **garageLightOn** — ceiling light status
- **garageObstructed** — safety sensor triggered (do NOT close if true)
- **garageLockState** — "locked" means physical remotes are disabled
- **vehicleState** — laser sensor detects parked car
- **vehicleDistance** — distance to vehicle in cm (laser)
- **motionDetected** — PIR motion sensor

## Control API

`POST http://<ip>/setgdo` with form data (IP from `config.json`):

| Field | Values | Effect |
|-------|--------|--------|
| `garageDoorState` | `1` = open, `0` = close | Opens or closes the door |
| `garageLightOn` | `1` = on, `0` = off | Toggles ceiling light |
| `garageLockState` | `1` = lock, `0` = unlock | Disables/enables physical remotes |

## Safety Rules

1. **Never close the door if `garageObstructed` is true.** Report the obstruction and stop.
2. **Always check status before opening/closing** to confirm current state and avoid unnecessary operations.
3. **Confirm before disabling remotes** — this locks out all physical remotes (wall button, car remotes).

## Helper Script

Use `scripts/garage.sh` for common operations:

```bash
# Status (human-readable)
bash scripts/garage.sh status

# Control
bash scripts/garage.sh open
bash scripts/garage.sh close
bash scripts/garage.sh light-on
bash scripts/garage.sh light-off
bash scripts/garage.sh lock-remotes
bash scripts/garage.sh unlock-remotes
```

## Notes

- **Not in Home Assistant** — HomeKit only allows single controller pairing. Paired to Apple Home for Siri control. Quinn uses the web API.
- **Vehicle laser:** Works well for detecting parked car. Distance reading varies by vehicle position.
- Update `config.json` with your device's IP, mDNS hostname, and MAC address after installation.
