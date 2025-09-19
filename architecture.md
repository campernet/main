# Campernet Architecture

![Campernet Banana Logo](https://github.com/campernet/main/blob/main/assets/nano-banana-pic3%20transparent%20background.png?raw=true)

**Campernet** is a modular, off-grid digital ecosystem composed of autonomous nodes—primarily single-board computers such as ESP32s and Raspberry Pis. Each module serves a distinct purpose (e.g. weather monitoring, gas level sensing, media playback), yet all are designed to operate independently or collaboratively, depending on context.

To maintain off-grid functionality, Campernet must provide its own Wi-Fi infrastructure. Rather than relying on a dedicated Wi-Fi server—which introduces a single point of failure—each module is capable of acting as a Wi-Fi access point (AP). On boot, a module performs the following:

1. Scans for a known Campernet SSID or beacon signature.
2. If found, it joins as a client (STA) and shares the network.
3. If not found, it initializes as a Wi-Fi access point, broadcasting the Campernet SSID.

This dynamic role-switching ensures resilience (no single module controls the network), autonomy (each module can operate standalone), and scalability (modules can be added or removed without reconfiguration). Modules periodically emit and listen for Campernet heartbeat packets (e.g. via UDP broadcast) to detect nearby APs. If a stronger AP is discovered, a module may demote itself to client mode to maintain network harmony.

To enable seamless discovery and integration, each module exposes a manifest—a simple JSON file served at a known endpoint (e.g. `/manifest.json`). This file describes the module’s identity, capabilities, and available data. For example:

```json
{
  "module": "weather",
  "type": "sensor",
  "version": "1.0",
  "sensors": ["temperature", "humidity", "pressure", "rain", "wind_speed", "wind_direction"],
  "endpoints": {
    "html": "/",
    "json": "/data.json"
  }
}
