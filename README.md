# GTW-08 Modbus gateway for Home Assistant (HACS)

Connect a heat pump or boiler from the BDR Thermea group (Remeha, Brötje, De Dietrich, Baxi) to Home Assistant through a GTW-08 Modbus gateway. This is the custom integration build of the `gtw08` integration, for testing through [HACS](https://hacs.xyz) before the integration lands in Home Assistant itself.

It reads temperatures, water pressure, status, heat output, the COP, energy counters, service state and board errors. Each heating zone becomes a thermostat, and the heating curve of each zone can be adjusted.

## Requirements

- Home Assistant 2026.10 or later. The integration shares its Modbus connection through Home Assistant's `modbus` integration, which gained that in 2026.9, and needs the modbus-connection release that 2026.10 ships.
- A GTW-08 Modbus gateway on the appliance's L-Bus.
- An RS-485 link from Home Assistant to the gateway's Modbus terminals: a USB to RS-485 adapter, or an RS-485 to Ethernet converter.

The gateway's factory settings are 9600 baud, no parity and unit ID 100. DIP switches 1 and 2 set the baud rate, DIP switches 3 and 4 set the parity, and the rotary dial adds its position to the unit ID.

## Installation

1. In HACS, open the menu in the top right corner and choose **Custom repositories**.
2. Add `https://github.com/AlCalzone/hacs-gtw08` with the type **Integration**.
3. Find **GTW-08 Modbus gateway** in HACS, download it, and restart Home Assistant.
4. Go to **Settings** > **Devices & services**, select **Add integration**, and choose **GTW-08 Modbus gateway**.
5. Pick how the gateway is connected:
   - **RS-485 adapter on this machine**: choose the serial port, baud rate and parity.
   - **RS-485 to Ethernet converter, Modbus TCP**: enter the converter's host and port.
   - **RS-485 to Ethernet converter, transparent**: enter the converter's host and port, baud rate and parity.

The unit ID sits under **More options**. The integration reads the appliance's control boards before it creates the entry.

## What you get

On the appliance device:

- **Outdoor temperature**, **Flow temperature**, **Return temperature**, **Water pressure**, **Status**, **Sub status**, **Relative power**, **Heat output** and **Pump speed**. Heat pumps add their own flow and return temperatures and the **Coefficient of performance**. Boilers add the **Flue gas temperature**.
- **Energy consumed** and **Heat delivered**, in total and for heating, hot water, cooling and backup. These go straight into the Energy dashboard.
- Binary sensors for what runs (heat pump, pump, central heating, hot water, cooling) and for problems (water pressure low, reset needed, service required, error).
- Diagnostic sensors for starts, operating hours, the service state and each control board's error code.

A device per zone with its flow temperature, setpoints, activity and operating mode. Zones that heat rooms get a thermostat: **Auto** follows the time program, **Heat** holds the manual temperature, and **Off** keeps frost protection only. Setting a temperature while on **Auto** switches the zone to **Heat**. Every zone has an **Operating mode** select, and heating zones have numbers for the heating curve.

The gateway's remote control of the heat demand is there too, as disabled entities: **Remote control**, **Remote heat demand**, **Remote power request** and **Remote temperature request**. While remote control is on, the appliance ignores its zones.

The integration polls every 30 seconds. The gateway fetches a value from the appliance the first time it is asked for it, so some entities can stay unavailable until the second poll.

## Reporting problems

Open an [issue](https://github.com/AlCalzone/hacs-gtw08/issues) and attach the diagnostics download from the integration's device page. It contains the raw registers the integration read, minus the serial number, host and serial port. The register map comes from the manufacturer's configuration manual 7854678 of September 2023 and has not been checked against a real appliance yet, so a raw dump is what a wrong value is tracked down with.

To see what the integration does, turn on debug logging:

```yaml
logger:
  logs:
    custom_components.gtw08: debug
```

## Development

This repository follows [integration_blueprint](https://github.com/ludeeus/integration_blueprint). The code in `custom_components/gtw08` and the tests in `tests/gtw08` are generated, so fix bugs at their source:

- [gtw08-modbus](https://github.com/AlCalzone/gtw08-modbus) is the device library. It is vendored as `custom_components/gtw08/gtw08_modbus`, so the integration needs no package from PyPI.
- [ha-core-gtw08](https://github.com/AlCalzone/ha-core-gtw08) is the integration as it is meant for Home Assistant core.

With both checked out next to this repository, rebuild and test:

```bash
scripts/setup
scripts/vendor.py
python3 -m pytest
```

`scripts/vendor.py` rewrites the library imports to relative ones, turns the manifest into a custom integration's, and resolves `strings.json` into `translations/en.json`. It keeps the version in the current manifest. The core integration targets core's `dev` branch, so the script renames the few things the released Home Assistant still calls differently, and records the test snapshots again against it. Review `git diff` after a run.

`scripts/develop` starts a Home Assistant instance with the integration loaded, configured by `config/configuration.yaml`.

## License

Apache License 2.0
