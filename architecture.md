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
This approach ensures self-description (modules announce their capabilities without hardcoding), inspectability (apps and other modules can query and adapt dynamically), and modularity (each module remains sovereign, yet discoverable). Some modules, like the weather station, contain multiple sensors. For example, a weather ESP32 might include a BME280 sensor (temperature, humidity, pressure), a rain gauge, and wind speed and direction sensors. While the BME280 provides three distinct readings, it is treated as one physical sensor with three logical outputs. This distinction simplifies wiring and failure handling while preserving data granularity.

In early stages, data will be served via standard HTTP and viewed in a browser. However, as Campernet grows, browser-based access becomes limiting. A dedicated Campernet app will offer autodiscovery of active modules via mDNS, UDP, or registry queries; dynamic UI based on available sensors and endpoints; structured data parsing via JSON; offline resilience and caching; and messaging and control across modules. This shift from browser to app reflects the architectural need for modular, inspectable APIs and adaptive interfaces.

Future considerations include a voice assistant, possibly based around Open Voice OS (OVOS) and a large language model such as TinyLlama via Llamafile. The access points will probably need some sort of password protection.
What we want to avoid is creeping back to someone else’s system for the sake of convenience. Home Assistant seems to be an admirable open source setup and there would be definite advantages to using meshtastic for messaging, but Campernet has its own ethos that we don’t want to dilute. The Campernet lanifesto has been published elsewhere but I will repeat it here:

## 📃 The Campernet Lanifesto

1. **There is no such thing as a necessary cookie.**  
   All cookies are unnecessary and most are unwanted. The only exception is those that remember passwords — and this task belongs to the Campernet browser, or app, which stores them locally, securely, and without drama. Ads, trackers, and surveillance are left at the gate.

2. **Every update must be uninstallable.**  
   Developers are imperfect and mistakes will happen. Rollback is not a luxury — it is a necessity.

3. **Offline is not broken.**  
   It is intentional. Campernet thrives without WAN. No “you’re not connected” errors — just making do with what resources you have.

4. **Every module must earn its place.**  
   No fluff, no bloat. If it doesn’t serve a purpose, tell a story or make people smile then it doesn’t belong.
