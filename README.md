#![HI-FAN ESPHome + Home Assistant overview](Hi-Fan%20BLE%20Home%20Assistant%20Integration.png)\n\n hi-fan-esphome

Reverse-engineered ESPHome integration for a Chinese BLE ceiling fan/light with Home Assistant.

This project started with a ceiling fan/light that could only be controlled by its original remote and the **Fan Light Elfin** Android app. By observing the BLE traffic and analyzing the app's behavior, the controller protocol and authentication sequence were reconstructed and implemented directly in ESPHome.

No Tuya gateway or original app is required for normal operation once ESPHome is configured.

## What works

- Light ON / OFF
- Brightness control
- Color temperature control (3000–5500 K on the tested device)
- Fan ON / OFF
- Six fan speeds
- Native night-light command
- Device → Home Assistant state synchronization from BLE notifications
- Home Assistant → device control
- Automatic BLE authentication
- Power Restore Guard: after an authenticated BLE session disconnects and reconnects, the light is forced OFF
- Manual HEX transmitter for protocol testing and reverse engineering

## Tested hardware

The reference setup uses:

- Chinese BLE ceiling fan/light compatible with **Fan Light Elfin**
- ESP32-C3
- ESPHome
- Home Assistant

The tested BLE interface is:

| Function | UUID |
| --- | --- |
| Service | `5746` |
| Write | `5701` |
| Read | `5702` |
| Notify | `5703` |

Visually similar fan/light controllers may use a different BLE protocol.

## Authentication

The tested controller requires authentication after every BLE connection.

The packet format discovered during reverse engineering is:

```text
00 FB 03 XX XX XX
         └────── last 3 bytes of the BLE MAC
```

Example:

```text
BLE MAC: AA:BB:CC:DD:EE:FF
AUTH:    00 FB 03 DD EE FF
```

The public ESPHome configuration therefore contains placeholders:

```yaml
substitutions:
  hi_fan_mac: "AA:BB:CC:DD:EE:FF"
  auth_b1: "0xDD"
  auth_b2: "0xEE"
  auth_b3: "0xFF"
```

See [docs/authentication.md](docs/authentication.md) for the discovery notes.

## Installation

Copy [esphome/hi-fan.yaml](esphome/hi-fan.yaml) into your ESPHome configuration directory.

Add your Wi-Fi credentials to `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
```

Then change `hi_fan_mac` and the three authentication bytes at the top of `hi-fan.yaml` to match your fan/light.

For a BLE MAC ending in `DD:EE:FF`:

```yaml
hi_fan_mac: "AA:BB:CC:DD:EE:FF"
auth_b1: "0xDD"
auth_b2: "0xEE"
auth_b3: "0xFF"
```

Compile and flash the configuration to the ESP32-C3 using ESPHome. After it joins Home Assistant, the configuration exposes the light, fan and night-light control.

## Known commands

Some of the confirmed commands from the tested controller are:

| Function | Packet |
| --- | --- |
| Light OFF | `0F 03 01 00` |
| Light ON | `0F 03 01 01` |
| Brightness | `00 0E 01 XX` |
| Color temperature | `00 0F 01 XX` |
| Night light | `00 13 02 05 05` |
| Fan OFF | `00 02 01 00` |
| Fan speed 1 | `00 02 01 0A` |
| Fan speed 2 | `00 02 01 14` |
| Fan speed 3 | `00 02 01 1E` |
| Fan speed 4 | `00 02 01 28` |
| Fan speed 5 | `00 02 01 32` |
| Fan speed 6 | `00 02 01 3C` |

Brightness and color-temperature values use a variable final byte. On the tested light, color-temperature raw value 0 corresponds to approximately 3000 K and 100 to approximately 5500 K.

More protocol information is in [docs/ble-protocol.md](docs/ble-protocol.md).

## Power Restore Guard

Many ceiling lights turn themselves ON after mains power is interrupted and restored. That is undesirable when the house is unattended.

The current configuration detects the loss of a previously authenticated BLE connection. After the controller reconnects and successfully authenticates, ESPHome sends a LIGHT OFF command.

This behavior is intentionally conservative: a BLE disconnect followed by reconnection can also trigger the guard even when the mains power did not actually fail.

The guard state is stored in RAM. If the ESP32 itself loses power at the same time as the ceiling light, the pending state is lost and this protection cannot act on that event.

## Original Fan Light Elfin application

The original **Fan Light Elfin** Android application was used as a reference while investigating interoperability.

The APK and decompiled application source are **not distributed by this project**. This repository contains only independently documented protocol behavior and the ESPHome implementation.

## Home Assistant wall controls

The BLE integration does not depend on any particular wall switch. In the reference installation, a separate three-button Zigbee scene switch is used through Home Assistant to control the ESPHome entities. That switch is optional and is not part of the BLE protocol.

## Compatibility

This project is based on one tested controller and is not an official vendor implementation.

If your device uses service `5746` and characteristics `5701` / `5703`, it may be related, but that alone does not guarantee compatibility. Check the BLE traffic before sending unknown commands.

If you successfully test another model, please open an issue and include the device model, BLE name, relevant UUIDs and observed packets.

## Project files

- [ESPHome configuration](esphome/hi-fan.yaml)
- [BLE protocol notes](docs/ble-protocol.md)
- [Authentication notes](docs/authentication.md)

## Disclaimer

This is an independent reverse-engineering project for interoperability and experimentation. It is not affiliated with or endorsed by the manufacturer of the ceiling fan/light or the developer of Fan Light Elfin.
