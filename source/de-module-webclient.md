# WebClient Module

The **DroneEngage WebClient** is a browser-based Ground Control Station (GCS). It lets you monitor and control drones from any modern web browser without installing desktop software.

## What it does

- Displays real-time telemetry, maps, and video streams.
- Sends commands to the drone (arm, mode change, mission upload, camera control).
- Plans missions, draws geo-fences, and manages swarms.
- Supports gamepad-based remote control over the internet.
- Runs in two modes:
  - **Cloud mode** — connect through the DroneEngage cloud server.
  - **WebPlugin / LAN mode** — local deployment for private networks.

## When to use it

Use the WebClient whenever you need a ground control station accessible from a laptop, tablet, or phone. It is the main user-facing interface of the DroneEngage ecosystem.

## Quick links

- [Web Client guide](de-web-client.rst)
- [Gamepad control](webclient-gamepad.rst)
- [Swarm control](webclient-swarm.rst)
- [UDP telemetry forwarding](webclient-udp-telemetry.rst)
- [Servo control](webclient-servo.rst)
- [WebPlugin / local deployment](webclient-web-plugin.rst.old)

## For developers

- **Repository**: `droneengage_webclient_react`
- **Stack**: React 19, Bootstrap 5, Leaflet maps, WebSocket, WebRTC.
- **Key files**: `src/index.js`, `src/js/js_andruav_auth.js`, `src/js/server_comm/js_andruav_ws.js`, `src/js/js_leafletmap.js`.
- **Configuration**: `public/config.json` controls endpoints, feature flags, and map providers.
- **WebConnector**: the `webconnector/` directory contains the local deployment helper.

See [Web Client Technicals](technicals/webclient/index) for deeper architecture and protocol details.
