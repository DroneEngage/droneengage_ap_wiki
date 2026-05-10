# Communication Server Internals

## Overview

The Communication Server (`droneengage_server`) is a Node.js application that exchanges communication messages between different units and web clients via WebSocket connections. It maintains a persistent WebSocket connection to the authentication server for coordination.

## Key Components

### Communication Server Manager (`server/js_comm_server_manager_client.js`)

Acts as a WebSocket client to the authentication server:

- **Connection**: Connects to `wss://s2s_ws_target_ip:s2s_ws_target_port`
- **Auto-reconnect**: Automatically reconnects with retry interval (2000ms) on connection loss
- **Message Handling**: Delegates incoming messages to registered handlers
- **Message Sending**: Provides `fn_sendMessage()` to send messages to the authentication server

#### Key Functions

- `fn_startServer()` - Starts the WebSocket connection to authentication server
- `fn_onOpen_Handler()` - Called when connection is established
- `fn_onClose_Handler()` - Handles connection close and triggers reconnection
- `fn_onMessage_Handler()` - Delegates incoming messages to `fn_onMessageReceived` callback
- `fn_sendMessage()` - Sends a message to the authentication server

### Communication Server (`server/js_andruav_comm_server.js`)

Manages the communication server logic and coordination with the authentication server.

#### Waiting Accounts

- `m_waitingAccounts` - Map of temporary login keys to pending login requests
- Keys expire after `CONST_WAIT_PARTY_TO_CONNECT_TIMEOUT` (10000ms)

#### Key Functions

- `isLoginExist(p_key)` - Checks if a temporary login key exists in waiting list
- `getLogin(p_key)` - Retrieves login request by temporary key
- `deleteLogin(p_key)` - Removes login request from waiting list
- `fn_addWaitingAccount(p_tempLoginKey, p_LoginRequest)` - Adds a login request to waiting list with timeout
- `fn_decryptAuthMessage(p_msg)` - Parses JSON message from authentication server
- `fn_generateLoginRequestReply(p_cmd)` - Generates reply to login request from authentication server
- `fn_AuthServerConnectionHandler()` - Called when authentication server connection is established
- `fn_handleLoginResponses(p_cmd)` - Processes login request from authentication server
- `fn_AuthServerMessagesHandler(p_msg)` - Routes messages from authentication server
- `fn_updateServerWatchdog()` - Sends server info to authentication server
- `fn_startServer()` - Initializes and starts the communication server

#### `fn_handleLoginResponses()`

Processes a login request from the authentication server:

1. Generates a temporary login key (UUID without dashes)
2. Stores the login request in `m_waitingAccounts[tempKey]`
3. Generates a reply with the temporary key and server connection details
4. Sends the reply to the authentication server via WebSocket

#### `fn_generateLoginRequestReply()`

Builds the JSON reply sent to the authentication server:

```json
{
  "c": "b",
  "d": {
    "r": "request-id",
    "e": 0,
    "g": "communication-server-host",
    "h": "communication-server-port",
    "f": "temporary-login-key"
  }
}
```

#### `fn_updateServerWatchdog()`

Sends server info to the authentication server:

```json
{
  "c": "a",
  "d": {
    "isOnline": true,
    "version": "server-version",
    "serverId": "server-id",
    "public_host": "public-host",
    "serverPort": "server-port",
    "accounts": ["account-key-1", "account-key-2"]
  }
}
```

This is sent periodically to keep the authentication server updated on the communication server status and served accounts.

### Chat Server (`server/chat_server/js_andruav_chat_server.js`)

Handles WebSocket connections from authenticated clients.

#### `fn_onConnect_Handler(p_ws, p_req)`

Main WebSocket connection handler:

1. Extracts parameters from the request URL
2. Validates the temporary login key
3. If valid, accepts the connection and adds the client to the appropriate account room

#### `fn_validateKey(p_params)`

Validates the temporary login key:

- Checks if the key exists in the URL parameters
- Validates the key format (alphanumeric, max length 200)
- Checks if the key exists in `m_waitingAccounts`
- Closes the connection if validation fails

#### `acceptConnection(v_loginTempKey, c_params, p_ws)`

Accepts a validated connection:

1. Retrieves the login request using the temporary key
2. Builds an onboard object with account ID, group ID, request ID, actor type, permissions
3. Deletes the temporary key from `m_waitingAccounts` (single-use)
4. Calls `_acceptConnection()` to add the client to the account room
5. Notifies the authentication server via `fn_onMessageOpened()`

#### `acceptLocalConnection(c_params, p_ws)`

Accepts a local connection (when `local_server_enabled=true`):

- Used for local server mode without authentication server
- Generates local account and group IDs
- Bypasses temporary key validation

## Constants (`js_constants.js`)

Key constants used in communication:

- `CONST_CS_CMD_INFO` - Server info command (`a`)
- `CONST_CS_CMD_LOGIN_REQUEST` - Login request command (`b`)
- `CONST_CS_CMD_LOGOUT_REQUEST` - Logout request command (`c`)
- `CONST_CS_ACCOUNT_ID` - Account ID field (`a`)
- `CONST_CS_GROUP_ID` - Group ID field (`b`)
- `CONST_CS_SENDER_ID` - Sender ID field (`s`)
- `CONST_CS_LOGIN_TEMP_KEY` - Temporary login key field (`f`)
- `CONST_CS_ERROR` - Error field (`e`)
- `CONST_CS_SERVER_PUBLIC_HOST` - Server host field (`g`)
- `CONST_CS_SERVER_PORT` - Server port field (`h`)
- `CONST_CS_REQUEST_ID` - Request ID field (`r`)

## Configuration (`server.config`)

Key configuration fields:

- `server_id` - Server identifier
- `server_ip` - Listening IP (default: `::`)
- `public_host` - Public host/IP as seen by clients
- `server_sid` - Unique server ID for multi-server deployments
- `server_port` - Listening port (default: 9966)
- `enable_SSL` - Enable SSL for client connections
- `s2s_ws_target_ip` - Authentication server IP for S2S connection
- `s2s_ws_target_port` - Authentication server port for S2S connection
- `ssl_key_file` - SSL private key file path
- `ssl_cert_file` - SSL certificate file path
- `allow_fake_SSL` - Allow fake SSL (for testing only)
- `ca_cert_path` - Custom CA certificate path
- `ignore_auth_server` - Ignore authentication server (for local mode)
- `local_server_enabled` - Enable local server mode

## Connection Flow

### Server Startup

1. Communication server starts
2. Connects to authentication server via WebSocket
3. Sends server info (watchdog) to authentication server
4. Authentication server marks the communication server as online

### Client Connection

1. Client authenticates with authentication server
2. Authentication server requests login reservation from communication server
3. Communication server generates temporary login key and stores in waiting list
4. Authentication server returns temporary key and connection details to client
5. Client connects to communication server WebSocket with temporary key
6. Communication server validates temporary key
7. Communication server accepts connection and adds client to account room
8. Temporary key is deleted (single-use)

## Security Considerations

- Temporary login keys are single-use and expire after timeout
- SSL/TLS for client connections (configurable)
- SSL/TLS for S2S connection to authentication server
- Key validation before accepting WebSocket connections
- Account room isolation prevents cross-account communication
- Active sender tracking prevents duplicate connections

## Related Documentation

- [Authentication Server Internals](de-server-technicals-authentication.md)
- [Authentication ↔ Communication Flow](de-server-technicals-auth-comm-flow.md)
