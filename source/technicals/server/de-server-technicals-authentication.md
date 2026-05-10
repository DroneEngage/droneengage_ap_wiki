# Authentication Server Internals

## Overview

The Authentication Server (`droneegnage_authenticator`) is a Node.js application that validates drone units and web clients, and returns connection details for the assigned communication server. It does not perform HTTP redirects; instead, it returns JSON with connection parameters.

## Key Components

### Routes (`routes/`)

#### Web Client Route (`js_router_web.js`)

Handles HTTP POST requests from web clients:

- **Endpoint**: `/web/login` (`CONST_WEB_FUNCTION + CONST_WEB_LOGIN_COMMAND`)
- **Parameters**: account name, access code, app group, app name, app version, extra
- **Actor Type**: `g` (GCS - Ground Control Station)
- **Validation**: Must be a GCS account (`mustGCS=true`)

#### Drone Agent Route (`js_router_agent.js`)

Handles HTTP POST requests from drone units:

- **Endpoint**: `/agent/al/` (`CONST_AGENT_FUNCTION + CONST_AGENT_LOGIN_COMMAND`)
- **Parameters**: account name, access code, app group, app name, app version, extra
- **Actor Type**: `d` (drone/agent)
- **Validation**: Must be an agent/drone account (`mustGCS=false`)

Both routes call `global.m_authServer.fn_newLoginCard()` with appropriate actor type and validation flags.

### Auth Server (`auth_server/js_auth_server.js`)

#### `fn_newLoginCard()`

Main login function that:

1. Validates input format (email, access code, group, app, version, extra)
2. Checks app version compatibility via `fn_checkAppVersion()`
3. Calls `fn_createLoginCard()` in session manager to create login card
4. Validates actor type (GCS vs Agent)
5. Selects a communication server via `fn_selectServerforAccount()`
6. Requests login reservation via `fn_requestCommunicationLogin()`

#### Session Manager (`auth_server/js_session_manager.js`)

#### `fn_createLoginCard()`

- Performs database login via `fn_do_loginAccount()`
- Generates session ID (`m_session_id`)
- Generates hashed account ID (`m_acc_id_hashed`)
- Stores login card in `m_loginCardList`
- Returns login card with account data, permissions, and group ID

#### `fn_generateLoginReplyToParty()`

Builds the JSON response sent to the client:

```json
{
  "e": 0,
  "sid": "session-id",
  "per": "permission",
  "prm": "permission2",
  "cs": {
    "f": "temporary-login-key",
    "g": "communication-server-host",
    "h": "communication-server-port"
  },
  "cm": "login-command"
}
```

### Communication Server Manager (`auth_server/js_comm_server_manager.js`)

#### `fn_selectServerforAccount()`

Selects a communication server for the client:

- Checks if the account is already served by a specific server
- If not, selects the current "best server" from available online servers
- Returns `null` if no server is available

#### `fn_requestCommunicationLogin()`

Requests the communication server to reserve a login slot:

- Generates a unique request ID (UUID)
- Stores the request in `m_waitingForServerLogin[requestId]`
- Sends a WebSocket message to the communication server with login request

#### `fn_handleLoginResponses()`

Handles the response from the communication server:

- Matches response by request ID
- Extracts temporary login key, server host, and port
- Updates the login card with server info
- Calls `fn_generateLoginReplyToParty()` to build final response

#### `fn_commServerMessageHandler()`

Handles all incoming messages from communication servers:

- Decrypts the message
- Routes based on command type:
  - `CONST_CS_CMD_INFO` (`a`) - Server info/registration
  - `CONST_CS_CMD_LOGIN_REQUEST` (`b`) - Login response

## Constants (`js_constants.js`)

Key constants used in authentication:

- `CONST_WEB_FUNCTION` - Web client route prefix
- `CONST_AGENT_FUNCTION` - Agent route prefix
- `CONST_WEB_LOGIN_COMMAND` - Web login command
- `CONST_AGENT_LOGIN_COMMAND` - Agent login command
- `CONST_SESSION_ID` - Session ID field
- `CONST_PERMISSION` - Permission field
- `CONST_PERMISSION2` - Extended permission field
- `CONST_COMM_SERVER` - Communication server info field
- `CONST_CS_CMD_INFO` - Server info command
- `CONST_CS_CMD_LOGIN_REQUEST` - Login request command
- `CONST_CS_ACCOUNT_ID` - Account ID field
- `CONST_CS_GROUP_ID` - Group ID field
- `CONST_CS_LOGIN_TEMP_KEY` - Temporary login key field
- `CONST_CS_SERVER_PUBLIC_HOST` - Server host field
- `CONST_CS_SERVER_PORT` - Server port field
- `CONST_CS_REQUEST_ID` - Request ID field

## Configuration (`server.config`)

Key configuration fields:

- `server_id` - Server identifier
- `server_ip` - Listening IP (default: 0.0.0.0)
- `server_port` - Listening port (default: 19408)
- `account_storage_type` - Account storage: single, file, or db
- `single_account_user_name` - Single account username (if type=single)
- `single_account_access_code` - Single account password (if type=single)
- `db_users` - JSON file path (if type=file)
- `s2s_ws_listening_ip` - WebSocket listening IP for communication servers
- `s2s_ws_listening_port` - WebSocket listening port for communication servers

## Account Storage

### Single Account

- Hardcoded account in server.config
- Useful for testing or single-user deployments

### File Storage

- JSON file with account entries
- Each account has: sid (account ID), pwd (password), isadmin, prm (permissions)
- Multiple accounts can share the same sid (same drone fleet)

### Database Storage

- MySQL database backend
- More scalable for production deployments

## Security Considerations

- Input validation for all parameters (email format, alphanumeric checks, version format)
- Actor type validation (GCS vs Agent)
- Permission checks before allowing login
- Communication server availability check before proceeding
- Session ID generation for tracking
- Hashed account IDs for privacy

## Related Documentation

- [Communication Server Internals](de-server-technicals-communication.md)
- [Authentication ↔ Communication Flow](de-server-technicals-auth-comm-flow.md)
