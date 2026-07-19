# Authentication ↔ Communication Server Flow

## Overview

This document describes the detailed interaction flow between the Authentication Server and Communication Server, including message formats, sequence diagrams, and the temporary login key lifecycle.

## Startup / Registration Sequence

Before any client can authenticate, the Communication Server establishes a persistent WebSocket connection to the Authentication Server and registers its presence.

### Sequence

```
Communication Server → Comm Server Manager Client: fn_startServer()
Comm Server Manager Client → Authentication Server: WebSocket connect (wss://s2s_ws_target_ip:s2s_ws_target_port)
Authentication Server → Comm Server Manager Client: Connection open
Comm Server Manager Client → Communication Server: fn_updateAuthServer()
Communication Server → Communication Server: fn_updateServerWatchdog()
Communication Server → Authentication Server: Send server info (c="a" CONST_CS_CMD_INFO)
Authentication Server → Auth Comm Server Manager: fn_handleServerInfo()
Auth Comm Server Manager → Auth Comm Server Manager: Mark server online, Update served account list
```

### Server Info Message

Communication server sends this message to register with the authentication server:

```json
{
  "c": "a",
  "d": {
    "isOnline": true,
    "version": "1.0.0",
    "serverId": "DE_Lap",
    "public_host": "airgap.droneengage.com",
    "serverPort": 9966,
    "accounts": ["account-key-1", "account-key-2"]
  }
}
```

- `c` - Command type: `a` for server info
- `d` - Data object containing server details
- `isOnline` - Server status
- `version` - Server version
- `serverId` - Unique server identifier
- `public_host` - Public host/IP as seen by clients
- `serverPort` - Port for client connections
- `accounts` - List of account keys currently served by this server

## Client Login Sequence

### Web Client Login

Web clients use the `/web/login` endpoint with actor type `g` (GCS).

### Drone Unit Login

Drone units use the `/agent/al/` endpoint with actor type `d` (agent/drone).

The flow is identical for both types, only the actor type differs.

### Full Login Sequence

```
Drone/Web Client → Auth HTTP Route: HTTP POST login (account, access code, group, app, version, extra)
Auth HTTP Route → Auth Server: fn_newLoginCard(actorType, ...)
Auth Server → Auth Server: Validate input format
Auth Server → Auth Server: fn_checkAppVersion()
Auth Server → Session Manager: fn_createLoginCard()
Session Manager → Session Manager: Database login
Session Manager → Session Manager: Generate session_id
Session Manager → Session Manager: Generate acc_id_hashed
Session Manager → Auth Server: loginCard
Auth Server → Auth Server: Validate actor permission
Auth Server → Auth Comm Server Manager: fn_selectServerforAccount()
Auth Comm Server Manager → Auth Comm Server Manager: Select best online server

[No server available]
  Auth Comm Server Manager → Auth Server: null
  Auth Server → Auth HTTP Route: Error 5 (SERVER_NOT_AVAILABLE)
  Auth HTTP Route → Drone/Web Client: JSON error

[Server selected]
  Auth Server → Auth Comm Server Manager: fn_requestCommunicationLogin()
  Auth Comm Server Manager → Auth Comm Server Manager: Generate requestId (UUID)
  Auth Comm Server Manager → Auth Comm Server Manager: Store in m_waitingForServerLogin
  Auth Comm Server Manager → Communication Server: WebSocket message (c="b" LOGIN_REQUEST, d.r=requestId, d.a=accountId, d.b=groupId, d.at=actorType)
  Communication Server → Communication Server: fn_handleLoginResponses()
  Communication Server → Communication Server: Generate temp key (UUID)
  Communication Server → Communication Server: fn_addWaitingAccount(tempKey, loginRequest)
  Communication Server → Auth Comm Server Manager: WebSocket reply (c="b", d.r=requestId, d.e=0, d.g=public_host, d.h=server_port, d.f=tempKey)
  Auth Comm Server Manager → Auth Comm Server Manager: fn_handleLoginResponses()
  Auth Comm Server Manager → Session Manager: fn_generateLoginReplyToParty()
  Session Manager → Auth Server: JSON reply (e=0, sid, per, prm, cs={f,g,h})
  Auth Server → Auth HTTP Route: login reply
  Auth HTTP Route → Drone/Web Client: JSON login reply
  Drone/Web Client → Chat WS Server: WebSocket connect (host=g, port=h, query/header includes f=tempKey)
  Chat WS Server → Chat WS Server: fn_onConnect_Handler()
  Chat WS Server → Communication Server: isLoginExist(f)

  [Key invalid/expired]
    Chat WS Server → Drone/Web Client: Close WebSocket

  [Key valid]
    Chat WS Server → Communication Server: getLogin(f)
    Chat WS Server → Communication Server: deleteLogin(f)
    Chat WS Server → Chat WS Server: acceptConnection()
    Chat WS Server → Chat WS Server: Add to account room
    Chat WS Server → Drone/Web Client: WebSocket accepted
```

## Message Formats

### Authentication Server → Communication Server (Login Request)

Sent when a client authenticates and the authentication server requests the communication server to reserve a login slot.

```json
{
  "c": "b",
  "d": {
    "r": "550e8400-e29b-41d4-a716-446655440000",
    "a": "hashed-account-id",
    "b": "1",
    "at": "d",
    "il": 5,
    "prm": "0xffffffff"
  }
}
```

- `c` - Command type: `b` for login request
- `d` - Data object
- `r` - Request ID (UUID) for matching response
- `a` - Hashed account ID
- `b` - Group ID (room ID)
- `at` - Actor type: `g` for GCS, `d` for drone
- `il` - Instance limit (max connections)
- `prm` - Permission mask

### Communication Server → Authentication Server (Login Response)

Sent in response to a login request, containing the temporary login key and connection details.

```json
{
  "c": "b",
  "d": {
    "r": "550e8400-e29b-41d4-a716-446655440000",
    "e": 0,
    "g": "airgap.droneengage.com",
    "h": 9966,
    "f": "a1b2c3d4e5f6g7h8i9j0"
  }
}
```

- `c` - Command type: `b` for login response
- `d` - Data object
- `r` - Same request ID from the request
- `e` - Error code: 0 for success
- `g` - Public host/IP for client connections
- `h` - Port for client connections
- `f` - Temporary login key (UUID without dashes)

### Authentication Server → Client (Login Reply)

Sent to the client after successful authentication, containing the communication server connection details.

```json
{
  "e": 0,
  "sid": "auth-session-id",
  "per": "0xffffffff",
  "prm": "0xffffffff",
  "cs": {
    "f": "a1b2c3d4e5f6g7h8i9j0",
    "g": "airgap.droneengage.com",
    "h": 9966
  },
  "cm": "login"
}
```

- `e` - Error code: 0 for success
- `sid` - Authentication session ID
- `per` - Permission (legacy)
- `prm` - Extended permission mask
- `cs` - Communication server info object
  - `f` - Temporary login key
  - `g` - Communication server host
  - `h` - Communication server port
- `cm` - Command name

## Temporary Login Key Lifecycle

### Generation

1. Authentication server sends login request to communication server
2. Communication server generates a UUID and removes dashes
3. Communication server stores the login request in `m_waitingAccounts[tempKey]`
4. Communication server sets a timeout (default: 10000ms) to auto-delete the key

### Delivery

1. Communication server returns the temporary key to authentication server
2. Authentication server includes the key in the JSON response to the client
3. Client receives the key along with connection details

### Usage

1. Client establishes WebSocket connection to communication server
2. Client includes the temporary key in the connection URL or headers
3. Communication server validates the key against `m_waitingAccounts`
4. If valid, communication server accepts the connection and deletes the key

### Expiration

- **Timeout**: Keys are automatically deleted after `CONST_WAIT_PARTY_TO_CONNECT_TIMEOUT` (10000ms)
- **Single-use**: Keys are deleted immediately after successful connection
- **Invalid keys**: Connections with invalid or expired keys are rejected

### Key Storage

Communication server maintains:

```javascript
m_waitingAccounts = {
  "a1b2c3d4e5f6g7h8i9j0": {
    "a": "hashed-account-id",
    "b": "group-id",
    "r": "request-id",
    "at": "actor-type",
    "prm": "permission-mask"
  }
}
```

## Error Handling

### No Communication Server Available

If no communication server is available:

1. Authentication server returns error code 5 (SERVER_NOT_AVAILABLE)
2. Client receives JSON error response
3. Client should retry authentication later

### Invalid Temporary Key

If the temporary key is invalid or expired:

1. Communication server closes the WebSocket connection
2. Client should re-authenticate to get a new key

### Login Request Timeout

If the communication server does not respond to a login request:

- Currently commented out in code
- Would return error 5 to the client
- Client should retry authentication

## Security Considerations

### Temporary Key Security

- Keys are randomly generated UUIDs (128-bit entropy)
- Keys are single-use, preventing replay attacks
- Keys expire quickly (10 seconds), reducing window for misuse
- Keys are deleted from memory after use, not persisted

### Server-to-Server Security

- S2S connection uses WebSocket with TLS (WSS)
- Communication server authenticates to authentication server
- Authentication server validates communication server identity
- Messages are encrypted in transit

### Client Connection Security

- Client connections use WebSocket with TLS (WSS) when enabled
- Temporary key validation prevents unauthorized connections
- Account room isolation prevents cross-account communication
- Active sender tracking prevents duplicate connections

## Related Documentation

- [Authentication Server Internals](de-server-technicals-authentication.md)
- [Communication Server Internals](de-server-technicals-communication.md)
