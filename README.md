# HiTE PRO Home Assistant Integration

[![GitHub Release][releases-shield]][releases]
[![hacs][hacsbadge]][hacs]
[![Project Maintenance][maintenance-shield]][user_profile]

Automatically discovers HiTE PRO gateways on your network and creates all their devices as MQTT entities in Home Assistant.

## How It Works

1. **mDNS Discovery**: Automatically finds HiTE PRO gateways on your local network
2. Fetches device configuration from the gateway's HTTP API
3. Parses the `defineVirtualDevice()` config
4. Publishes MQTT Auto-Discovery payloads to HA's broker
5. HA's built-in MQTT integration creates switches, lights, sensors, and binary sensors
6. State flows directly: **Gateway → MQTT Broker → Home Assistant** (no middleman)

## Prerequisites

- Home Assistant **2024.1** or newer
- The **MQTT** integration configured — the broker must be shared with your HiTE PRO gateway
- The **Zeroconf** integration enabled (included by default in HA)

## Installation

### HACS (Recommended)

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=illmouse&repository=hass-hitepro-integration&category=integration)

1. Install [HACS][hacs-download] if you don't have it yet
2. Click the button above or search for `HiTE PRO` in HACS
3. Click **Download**
4. Restart Home Assistant

### Manual

1. Copy the `custom_components/hitepro/` directory to your HA config:
   ```
   /config/custom_components/hitepro/
   ```
2. Restart Home Assistant

## Configuration

### Automatic (mDNS Discovery)

If your HiTE PRO gateway is on the same network, it will appear automatically:

1. Go to **Settings → Devices & Services**
2. A **"HiTE PRO Gateway"** discovery card will appear
3. Click **Configure** → review the auto-filled URL and API key → **Submit**

The gateway advertises itself via mDNS as `_hitepro._tcp` on port 80.

### Manual

[![Add Integration][add-integration-badge]][add-integration]

1. Click the button above, or go to **Settings → Devices & Services → Add Integration**
2. Search for **"HiTE PRO"**
3. Enter your gateway URL and API key:
   - **Gateway URL**: `http://192.168.x.x/mqtt/` (LAN) or your remote URL
   - **API Key**: The `key` parameter from your gateway
4. Click **Submit** — the integration validates the connection before saving

## Switches vs Lights

By default, all relay channels are created as **switches** in Home Assistant. You can configure specific channels to appear as **lights** instead:

1. Go to **Settings → Devices & Services → HiTE PRO → Configure**
2. The **Light devices** dropdown lists all switch-type channels fetched from your gateway
3. Select the channels that control lights (e.g. `Relay-4S_9F4CAE93_1`)
4. Click **Submit** — the integration will recreate those entities as MQTT lights

Lights created this way support on/off commands (they do not support brightness dimming, since the gateway only provides relay control). To revert a light back to a switch, simply deselect it from the list.

## Auto-Refresh

- Devices are refreshed every **5 minutes** by default (minimum 60 seconds)
- Change the interval in **Settings → Devices & Services → HiTE PRO → Configure**
- Manual refresh: call the `hitepro.refresh_devices` service

## Device Mapping

| HiTE PRO Type | Home Assistant Domain | Example |
|---|---|---|
| `switch` | `mqtt.switch` | Реле (Relay) |
| `range` | `mqtt.light` | Dimmer (brightness) |
| `temperature` | `mqtt.sensor` | Smart-Air temperature |
| `rel_humidity` | `mqtt.sensor` | Smart-Air humidity |
| `alarm` | `mqtt.binary_sensor` | Door checker, motion |
| `pushbutton` | `mqtt.binary_sensor` | Leak detector |
| `text` | `mqtt.sensor` | Illumination % |
| `rgb` | `mqtt.light` | RGBW controller |

## MQTT Topics

State and commands follow Wiren Board convention:

- **State**: `/devices/hite-pro/controls/{control_id}`
- **Command**: `/devices/hite-pro/controls/{control_id}/on`
- **Discovery**: `homeassistant/{domain}/{entity_id}/config`

## Troubleshooting

Enable debug logging:

[![Logging service][ha-service-badge]][ha-service]

On the entry page, paste:

```yaml
service: logger.set_level
data:
  custom_components.hitepro: debug
```

Then check the logs:

[![Home Assistant Logs][ha-logs-badge]][ha-logs]

## Removing the Integration

1. Go to **Settings → Devices & Services**
2. Click the three-dot menu on the HiTE PRO card → **Delete**
3. All MQTT discovery entries are removed automatically

## Files

```
custom_components/hitepro/
├── __init__.py          # Setup, refresh, service handler
├── config_flow.py       # Config, zeroconf discovery & options flow
├── const.py             # Constants
├── discovery.py         # Config parser & MQTT discovery
├── icons/
│   └── icon.png         # Integration icon
├── manifest.json        # Integration metadata (zeroconf, dependencies)
├── services.yaml        # Service definitions
├── strings.json         # UI strings
└── translations/
    ├── en.json           # English translations
    └── ru.json           # Russian translations
```

[add-integration]: https://my.home-assistant.io/redirect/config_flow_start/?domain=hitepro
[add-integration-badge]: https://my.home-assistant.io/badges/config_flow_start.svg
[hacs]: https://hacs.xyz
[hacs-download]: https://hacs.xyz/docs/setup/download
[hacsbadge]: https://img.shields.io/badge/HACS-Default-blue.svg?style=flat
[ha-logs]: https://my.home-assistant.io/redirect/logs
[ha-logs-badge]: https://my.home-assistant.io/badges/logs.svg
[ha-service]: https://my.home-assistant.io/redirect/developer_call_service/?service=logger.set_level
[ha-service-badge]: https://my.home-assistant.io/badges/developer_call_service.svg
[maintenance-shield]: https://img.shields.io/badge/maintained-yes-green.svg?style=flat
[releases-shield]: https://img.shields.io/github/release/illmouse/hass-hitepro-integration.svg?style=flat
[releases]: https://github.com/illmouse/hass-hitepro-integration/releases
[user_profile]: https://github.com/illmouse