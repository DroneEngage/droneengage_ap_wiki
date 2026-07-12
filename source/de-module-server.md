# Communication Server

The **Communication Server** is the real-time messaging backbone of DroneEngage. It relays WebSocket messages between drone units and Ground Control Stations, and it can form a server-to-server mesh for scalable deployments.

## What it does

- Accepts persistent WebSocket connections from drone units and GCS clients.
- Routes messages by group or individual target.
- Provides server-to-server (S2S) relay for multi-region or redundant setups.
- Offers a UDP proxy for MAVLink and other UDP protocols.
- Persists tasks and, optionally, message history via MySQL.

## Deployment modes

- **Standalone** — single server for local or small deployments.
- **Child server** — connects to a parent server for relay.
- **Parent (super) server** — accepts child connections and forwards messages across the mesh.

## Quick links

- [Server installation overview](srv-index.rst)
- [Communication Server setup](srv-communication.rst)
- [S2S authentication](srv-authentication.rst)
- [Air-gapped installation](srv-install-airgap.rst)

## For developers

- **Repository**: `droneengage_server`
- **Runtime**: Node.js 18+.
- **Stack**: Express, `ws`, MySQL2, Ed25519 S2S auth.
- **Key files**:
  - `server.js` — main entry point.
  - `server/js_andruav_comm_server.js` — WebSocket server.
  - `server/chat_server/js_chat_routing.js` — message routing.
  - `server/server_to_server/js_s2s_auth.js` — Ed25519 S2S authentication.
- **Configuration**: `server.config` (JSON, supports `--config` override).

See [Server technicals](technicals/server/index) for routing, S2S relay, and configuration details.
