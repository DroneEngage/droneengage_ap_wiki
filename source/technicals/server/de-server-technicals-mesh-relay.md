# Mesh Communication Server Relay

This document describes the message routing and propagation logic in the server-to-server relay system used by the DroneEngage Communication Server.

## Architecture Overview

```
                    ┌─────────────────────┐
                    │    Super Server     │
                    │  (ParentCommServer) │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌────────────┐   ┌────────────┐   ┌────────────┐
       │  Child A   │   │  Child B   │   │  Child C   │
       │(ChildComm) │   │(ChildComm) │   │(ChildComm) │
       └─────┬──────┘   └─────┬──────┘   └─────┬──────┘
             │                │                │
        Local Units      Local Units      Local Units
        (WS Clients)     (WS Clients)     (WS Clients)
```

## Key Components

### Files

- **`server/chat_server/js_andruav_chat_server.js`**
  - Main local and external message routing logic.
- **`server/server_to_server/js_parent_comm_server.js`**
  - Super server; accepts child communication-server connections.
- **`server/server_to_server/js_child_comm_server.js`**
  - Child server; connects to a parent/super communication server.

### Core Functions

- **`fn_parseMessage()`**
  - **Location:** `js_andruav_chat_server.js`
  - **Purpose:** Handles messages from local WebSocket clients.
- **`fn_parseExternalMessage()`**
  - **Location:** `js_andruav_chat_server.js`
  - **Purpose:** Handles messages received from relay servers and forwards them further in the relay tree.
- **`forwardMessage()`**
  - **Location:** `js_andruav_chat_server.js`
  - **Purpose:** Forwards local messages to parent or child relay servers.
- **`forwardExternalMessage()`**
  - **Location:** `js_andruav_chat_server.js`
  - **Purpose:** Forwards external relay messages further in the tree with sender exclusion.
- **`getServerOriginID()`**
  - **Location:** `js_andruav_chat_server.js`
  - **Purpose:** Returns the configured `server_id` used for loop prevention.
- **`ParentCommServer.forwardMessage()`**
  - **Location:** `js_parent_comm_server.js`
  - **Purpose:** Broadcasts a message to connected child servers, optionally excluding the sender.
- **`ChildCommServer.forwardMessage()`**
  - **Location:** `js_child_comm_server.js`
  - **Purpose:** Sends a message from a child server to its parent server.

## Data Flow

### 1. Local Client to Relay Propagation

When a local WebSocket client sends a message:

```
Local Client (WebSocket)
       │
       ▼
fn_onWsMessage()
       │
       ▼
fn_parseMessage()
       │
       ├──► Local delivery
       │       ├── send_message_toMyGroup()
       │       ├── send_message_toMyGroup_GCS()
       │       ├── send_message_toMyGroup_Agent()
       │       └── send_message_toTarget()
       │
       └──► forwardMessage()
               │
               ├──► Parent server if enable_super_server=true
               └──► Child/parent relay socket if enable_persistant_relay=true
```

Code path:

1. `fn_onWsMessage()` receives the WebSocket message.
2. `fn_parseMessage(p_ws, p_message, p_isBinary)` parses the message.
3. The server injects the permission field `p` from `p_ws.m_loginRequest.m_prm`.
4. The message is delivered to local clients according to `ty` and `tg`.
5. `forwardMessage()` propagates eligible messages to relay servers.

### 2. External Relay Server to Tree Propagation

When a message arrives from a relay server:

```
Parent or Child Relay Server
       │
       ▼
fn_parseExternalMessage()
       │
       ├──► Check _origin for loop prevention
       │
       ├──► Local delivery
       │       ├── fn_sendToAll()
       │       ├── fn_sendToAllGCS()
       │       ├── fn_sendToAllAgent()
       │       └── fn_sendTIndividualId()
       │
       └──► forwardExternalMessage()
               │
               ├──► Parent server (grandparent) if enable_persistant_relay=true
               └──► Children (excluding sender child) if enable_super_server=true
```

Code path:

1. `ParentCommServer` receives a child message, or `ChildCommServer` receives a parent message.
2. The relay wrapper calls `fn_parseExternalMessage(p_message, p_isBinary, p_source_ws)`.
3. The chat server checks the `_origin` field for loop prevention.
4. If the message originated from this server, it is ignored.
5. Otherwise, the message is delivered to local clients.
6. `forwardExternalMessage()` is called to propagate the message further in the relay tree:
   - If in child mode, forwards to parent (grandparent) for upward propagation.
   - If in parent mode, forwards to children, excluding the source child to prevent sending back to sender.
7. The `_origin` field prevents loops when messages bounce back to their origin server.

## Loop Prevention

### Server Origin ID

Each communication server should have a unique `server_id` in `server.config`. The chat server uses this value as the relay origin ID:

```javascript
function getServerOriginID() {
    return global.m_serverconfig?.m_configuration?.server_id || 'unknown';
}
```

### Origin Injection

`forwardMessage()` injects `_origin` before forwarding the message to another server:

```javascript
if (!v_jmsg._origin) {
    v_jmsg._origin = getServerOriginID();
}
```

This is done only on forwarded messages. Local-only processing does not require `_origin`.

### Origin Check

`fn_parseExternalMessage()` drops messages that came back to their origin server:

```javascript
if (v_jmsg._origin === getServerOriginID()) {
    return;
}
```

This prevents a child server from processing its own message when the parent broadcasts it back to all connected children.

## Text and Binary Message Handling

The propagation logic supports both text and binary WebSocket messages.

### Text Messages

For text messages, the server parses the JSON message, injects or checks fields, and serializes it again when forwarding.

### Binary Messages

For binary messages, the message format is:

```
JSON header + null byte + binary payload
```

When forwarding binary messages, `forwardMessage()` parses and modifies only the JSON header. The binary payload is preserved when reconstructing the message.

## Routing Types

The `ty` field determines the message routing behavior:

- **`g`**
  - **Constant:** `CONST_WS_MSG_ROUTING_GROUP`
  - **Behavior:** Group broadcast.
- **`i`**
  - **Constant:** `CONST_WS_MSG_ROUTING_INDIVIDUAL`
  - **Behavior:** Targeted or filtered delivery.
- **`s`**
  - **Constant:** `CONST_WS_MSG_ROUTING_SYSTEM`
  - **Behavior:** System command handling; local command path.

## Target Field Values

The `tg` field can contain a specific sender ID or a reserved broadcast target:

- **`_GCS_`**
  - **Constant:** `CONST_WS_SENDER_ALL_GCS`
  - **Meaning:** All GCS clients.
- **`_GD_`**
  - **Constant:** `CONST_WS_SENDER_ALL`
  - **Meaning:** All units, including GCS and agents.
- **`_AGN_`**
  - **Constant:** `CONST_WS_SENDER_ALL_AGENTS`
  - **Meaning:** All drone agents.
- **Any other non-empty value**
  - **Meaning:** Specific unit/sender ID.

## Local Routing Behavior

For local messages handled by `fn_parseMessage()`:

- GCS group messages are delivered to the local group.
- Drone group messages are delivered to local GCS clients.
- `_GCS_` targets local GCS clients.
- `_GD_` targets all local clients.
- `_AGN_` targets local drone agents.
- One-to-one messages are first attempted locally; if the target is not found, they are forwarded to relay servers.
- System messages are handled by `onSystemMessage()` and are not forwarded through the relay path.

## External Routing Behavior

For external messages handled by `fn_parseExternalMessage()`:

- Group messages are delivered to all local clients and propagated in the relay tree.
- `_GCS_` targets local GCS clients and is propagated in the relay tree.
- `_GD_` targets all local clients and is propagated in the relay tree.
- `_AGN_` targets local drone agents and is propagated in the relay tree.
- No target (default broadcast) targets all local clients and is propagated in the relay tree.
- Specific targets (one-to-one messages) are delivered locally if found but are NOT propagated in the relay tree.
- Only broadcast messages are re-forwarded to other relay servers to enable tree-like propagation:
  - Parent mode: Forwards to children, excluding the sender child.
  - Child mode: Forwards to parent (grandparent) for upward propagation.
- Loop prevention via `_origin` prevents messages from bouncing back to their origin.

## Configuration Flags

- **`enable_super_server`**
  - Enables parent/super-server mode and allows child communication servers to connect.
- **`enable_persistant_relay`**
  - Enables child relay mode and connects this server to a parent server.
- **`server_id`**
  - Unique identifier used in `_origin` for loop prevention.
- **`s2s_ws_target_ip`**
  - Parent server address for child relay mode.
- **`s2s_ws_target_port`**
  - Parent server port for child relay mode.

## Summary Table

- **Local WebSocket client**
  - **Handler:** `fn_parseMessage()`
  - **Local delivery:** Yes
  - **Relay forward:** Yes, for eligible messages
  - **Loop handling:** Injects `_origin` in `forwardMessage()`
- **Parent relay server**
  - **Handler:** `fn_parseExternalMessage()`
  - **Local delivery:** Yes
  - **Relay forward:** Yes, to children (excluding sender) and to parent (grandparent)
  - **Loop handling:** Drops messages whose `_origin` matches local `server_id`
- **Child relay server**
  - **Handler:** `fn_parseExternalMessage()`
  - **Local delivery:** Yes
  - **Relay forward:** Yes, to parent (grandparent)
  - **Loop handling:** Drops messages whose `_origin` matches local `server_id`

## Validation Notes

This document was reviewed against the current server implementation:

- `server/chat_server/js_andruav_chat_server.js`
- `server/server_to_server/js_parent_comm_server.js`
- `server/server_to_server/js_child_comm_server.js`

The documented behavior matches the current implementation:
- Local client messages are forwarded to relay servers via `forwardMessage()`.
- External relay messages are delivered locally and then re-forwarded via `forwardExternalMessage()` to enable tree-like propagation.
- Parent servers forward to children (excluding the sender child) and to parent (grandparent) if configured.
- Child servers forward to parent (grandparent) if configured.
- Loop prevention via `_origin` prevents messages from bouncing back to their origin server.
