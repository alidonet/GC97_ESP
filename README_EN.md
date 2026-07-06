# GC97 ESP8266

[Russian version](README.md)

Firmware for ESP8266 that turns the GC97 coulometer into a network device with a web interface, JSON API, MQTT, and Home Assistant integration.

![GC97 + ESP8266](manual/GC97_ESP.jpg)

[Download the ready-to-flash firmware](bin/GC97_ESP.bin)
Old firmware binaries are stored in [bin/previous](bin/previous/).

## Features

- GC97 data reading over TTL UART;
- web interface with live readings, settings, reboot, and OTA update;
- access point mode for first-time setup or recovery access;
- connection to a 2.4 GHz Wi-Fi network;
- mDNS and SSDP for device discovery in the local network;
- MQTT data publishing;
- Home Assistant MQTT Discovery;
- JSON API for direct data access;
- status indication with the onboard LED.

The firmware is intended for ESP8266 boards such as Wemos D1 mini, NodeMCU, and compatible boards. ESP32 is not supported yet.

## Wiring

GC97 is connected to ESP8266 over UART. Baud rate: `19200`.

Minimal wiring:

| GC97 | ESP8266 |
| --- | --- |
| TX | RX / GPIO3 |
| RX | TX / GPIO1 |
| GND | GND |

If no data appears, first try swapping RX and TX. A common ground is required.

ESP8266 uses 3.3 V logic levels. If the GC97 UART output is 5 V, use a level shifter or a simple voltage divider on the `GC97 TX -> ESP RX` line.

![ESP wiring](manual/wiring_GC97_ESP.jpg)

GC97 connection points:

![GC97 connection points](manual/wiring_GC97_points.jpg)

Power the ESP through USB, through the 5 V/VIN pin of your board, or from a stable 3.3 V supply if your board supports it. Follow the power requirements of your specific ESP8266 board.

## First Start

1. Flash `bin/GC97_ESP.bin` to the ESP8266 at address `0x000000`.
2. Reboot the board.
3. Connect to the open Wi-Fi network named `GC97-XXXXXX`.
4. Open `http://192.168.4.1`.
5. Open `Settings` and enter your home Wi-Fi credentials.
6. Save the settings and reboot the ESP.
7. Open the device using the IP address assigned by your router DHCP server.

![Main screen](manual/manual_web_main.png)

## Settings

The web interface provides:

- Wi-Fi and fallback access point settings;
- Wi-Fi network scanning;
- MQTT broker settings;
- GC97 polling period;
- MQTT publishing only when values change;
- Home Assistant Discovery option;
- OTA enable/disable option;
- onboard LED mode.

![Wi-Fi settings](manual/manual_options_wifi.png)

![Network scan](manual/manual_scan_networks.png)

![MQTT settings](manual/manual_options_mqtt.png)

![GC97 and ESP settings](manual/manual_options_misc.png)

## MQTT

Default base MQTT topic: `gc97`.

Main topics:

- `gc97/meter_json` - coulometer data;
- `gc97/esp_json` - ESP status and network parameters;
- `gc97/state_json` - short device state;
- `gc97/command` - control commands;
- `gc97/option/ask_timeout` - polling period control.

To reboot the device through MQTT, publish `restart` to `gc97/command`.

When Home Assistant Discovery is enabled, the device automatically creates entities for GC97 and ESP.

![Home Assistant devices](manual/manual_hass_devices.png)

![GC97 in Home Assistant](manual/manual_hass_gc97.png)

![ESP in Home Assistant](manual/manual_hass_esp.png)

## JSON API

Data can be read with regular HTTP requests:

- `http://<ip>/get/all.json` - full data package;
- `http://<ip>/get/gc97.json` - GC97 data;
- `http://<ip>/get/esp.json` - ESP status;
- `http://<ip>/get/state.json` - short state;
- `http://<ip>/get/device.json` - device description;
- `http://<ip>/get/logs.json` - logs.

![JSON data](manual/manual_json.png)

## OTA Update

If the device is already connected to Wi-Fi and OTA is enabled in settings, the firmware can be updated from the web interface with the `Upgrade` button.

You can also upload the firmware with:

```bash
curl -F "image=@GC97_ESP.bin" http://<ip>/update
```

After the update, the device will reboot. Settings should be preserved as long as the same flash memory layout is used.

## LED Indication

On startup, the ESP flashes a short LED sequence. During normal operation, status indication repeats roughly every 30 seconds:

- 1 flash - normal operation;
- 2 flashes - Wi-Fi problem;
- 3 flashes - MQTT problem;
- 4 flashes - no valid data from GC97.

The `Turn on LED when ready` option reverses the onboard LED behavior in the ready state.

## Service Commands

- `http://<ip>/cmd/reset` - reboot ESP;
- `http://<ip>/cmd/factory` - reset settings to factory defaults.

Factory reset removes saved Wi-Fi and MQTT settings. After that, the device will start the `GC97-XXXXXX` access point again.

## Finding the Device on the Network

After connecting to your home Wi-Fi network, the device can be found:

- by IP address in the router DHCP table;
- through Windows network discovery if SSDP works in your network;
- by the mDNS name `GC97-XXXXXX.local` if mDNS is supported by your system.

If the option is enabled, the access point automatically turns off 5 minutes after a successful Wi-Fi connection.
The interface is also adapted for mobile devices.

![Network device](manual/manual_gc97_network.png)

![SSDP properties](manual/manual_ssdp.png)

## Known Notes

- GC97 and ESP must share a common ground.
- ESP8266 is sensitive to power quality, especially during Wi-Fi transmission.
- If OTA is disabled or the device is not connected to Wi-Fi, remote update through `/update` will not be available.
- If Wi-Fi settings are wrong, the device should start its own access point so the configuration can be recovered.

## Feedback

If you find a bug or want to suggest an improvement, please create an issue in the project repository.
