# Authenticator

The **Authenticator** is the central authentication and authorization server for DroneEngage. It issues accounts, access codes, and session tokens, and it authenticates Communication Servers using Ed25519 server-to-server (S2S) cryptography.

## What it does

- Registers users and generates Access Codes.
- Authenticates WebClient logins and unit connections.
- Manages registered Communication Servers.
- Provides an admin web interface for users and servers.
- Exposes a WebSocket S2S endpoint for server authentication.

## Components

The Authenticator runs three services simultaneously:

1. **API Server** (default port `19408`) — REST API for login, register, agent routes.
2. **Views Server** (default port `8089`) — EJS-based admin web interface.
3. **S2S WebSocket Server** (default port `19001`) — Ed25519 challenge-response auth for servers.

## Quick links

- [Authentication setup](srv-authentication.rst)
- [Server installation overview](srv-index.rst)
- [S2S key generation](https://github.com/DroneEngage/droneegnage_authenticator/blob/main/scripts/gen_s2s_keys.sh)

## For developers

- **Repository**: `droneegnage_authenticator`
- **Runtime**: Node.js 18+.
- **Stack**: Express, EJS, SQLite3, Helmet, CORS, CSRF, rate limiting.
- **Key files**:
  - `server.js` — spawns all three services.
  - `auth_server/js_auth_server.js` — main auth controller.
  - `auth_server/js_account_manager.js` — user CRUD.
  - `auth_server/js_s2s_auth.js` — Ed25519 S2S auth.
  - `routes/js_router_admin.js` — admin routes.
- **Configuration**: `server.config` (JSON, supports `--config` override).
- **Database**: SQLite with migrations in `database/migrations/`.

See [Authenticator technicals](technicals/server/index) for the authentication flow, database schema, and API details.
