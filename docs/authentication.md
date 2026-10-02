# BLE authentication

## Discovery

The authentication mechanism was identified while analyzing the original Android application used with the tested ceiling fan/light, **Fan Light Elfin** (package name `com.warpfuture.wfiot.fan`).

The application constructs a six-byte authentication packet using the last three bytes of the device's BLE MAC address:

```text
00 FB 03 XX XX XX
         └────── last 3 bytes of BLE MAC
```

Example:

```text
BLE MAC: AA:BB:CC:DD:EE:FF
AUTH:    00 FB 03 DD EE FF
```

The packet is written to characteristic `5701` after connecting. On the tested controller, successful authentication is echoed on notification characteristic `5703`.

## ESPHome behavior

The example configuration waits briefly after BLE connection and then sends the authentication packet. Normal control commands should only be considered usable after successful authentication.

During reverse engineering, the tested controller returned:

```text
00 FB 03 00 00 00
```

when authentication failed.

## Why the original APK is not included

The original **Fan Light Elfin** APK is proprietary third-party software and is **not distributed by this project**.

The application was used only as a reference while investigating interoperability with hardware owned by the project author. This repository documents the independently observed BLE behavior required to communicate with the device; it does not redistribute the application or its decompiled source code.

## Compatibility warning

This is an observed protocol from the tested hardware/software combination, not an official vendor specification. Other versions of Fan Light Elfin or visually similar ceiling fan controllers may use different authentication or BLE commands.

If you test another device successfully, please open an issue or contribute your findings.
