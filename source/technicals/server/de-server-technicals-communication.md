# Communication Server Technicals

## Overview

DroneEngage Communication Server is the real-time messaging backbone for the DroneEngage/Andruav drone management ecosystem. It handles WebSocket-based communication between drone units and Ground Control Stations (GCS), implements server-to-server mesh relay for scalable message propagation, and supports message routing with group and individual targeting using Ed25519 cryptographic authentication.

## Tech Stack

- **Runtime**: Node.js >= 18
- **Web Framework**: Express.js
- **Database**: MySQL2
- **Real-time**: WebSocket (ws library)
- **Authentication**: Ed25519 S2S authentication
- **UDP**: udp-packet for UDP proxy functionality
- **Utilities**: lodash, moment, uuid, randomstring, jspack

## Architecture

Three operational modes:

### 1. Standalone Mode
- Independent server for local communication
- No S2S relay required
- Suitable for small-scale operations

### 2. Child Server Mode
- Connects to parent server for message relay
- Receives messages from parent
- Forwards local messages to parent
- Part of distributed mesh network

### 3. Parent (Super) Server Mode
- Accepts child connections
- Forwards messages between children
- Implements mesh relay for scalability
- Central hub in distributed deployment

## Core Components

### WebSocket Server
- Main communication channel for units and GCS
- Handles client connections with temporary key validation
- Manages account rooms for message routing
- Maintains active sender lists

### S2S Relay
- Server-to-server mesh network for message propagation
- Ed25519 cryptographic authentication
- Loop prevention using path tracking
- Message routing between parent and child servers

### UDP Proxy
- Handles UDP packet forwarding
- Kernel buffer size checking and adjustment
- Fixed port configuration option
- Used for MAVLink and other UDP protocols

### Chat System
- Message routing and room management
- Group and individual targeting
- Active sender tracking
- Connection lifecycle management

## Key Files

### Core Server Files
- `server.js` - Main entry point, initializes all servers
- `server.config` - JSON configuration (supports --config override)
- `package.json` - Dependencies and scripts
- `js_constants.js` - Message types, routing constants

### Communication Server Components
- `server/js_andruav_comm_server.js` - Main WebSocket server
- `server/js_s2s_auth.js` - Ed25519 S2S authentication
- `server/js_udp_proxy.js` - UDP packet proxy
- `server/js_andruavTasks_v2.js` - Task management

### Chat System
- `server/chat_server/js_andruav_chat_server.js` - Chat system singleton
- `server/chat_server/js_chat_routing.js` - Message routing logic
- `server/chat_server/js_chat_connection.js` - Connection management
- `server/chat_server/js_chat_relay.js` - S2S message relay

### S2S Components
- `server/server_to_server/js_parent_comm_server.js` - Parent server S2S
- `server/server_to_server/js_child_comm_server.js` - Child server S2S

### Helpers
- `helpers/hlp_args.js` - Argument parsing utilities
- `helpers/hlp_strings.js` - String manipulation utilities
- `helpers/hlp_validation.js` - Input validation
- `helpers/hlp_colors.js` - Console color formatting

## Configuration

Configuration is centralized in `server.config` (JSON format). Key sections:

### Server Identity
- `server_id` - Unique server identifier
- `server_ip` - Listening IP (default: `::`)
- `server_port` - Listening port (default: 9966)
- `public_host` - Public host/IP as seen by clients
- `server_sid` - Unique server ID for multi-server deployments

### Database Connection
- MySQL credentials
- Connection pool configuration
- Database name and host

### S2S Authentication
- Ed25519 private key
- Trusted public keys
- Authentication server connection details

### SSL/TLS
- SSL certificate paths
- SSL private key path
- CA certificate path
- SSL enable/disable flags

### Server Roles
- `enable_super_server` - Enable parent server mode
- `enable_persistant_relay` - Enable persistent relay
- Parent/child server connection details

### Logging
- Log level configuration
- Log file paths
- Log rotation settings

### Memory Management
- `memory_max` - Memory limit in MB
- Auto-restart on limit exceeded

## Coding Standards

### Logging
- Use `global.m_logger` for logging (if enabled)
- Check `global.m_logger` existence before use
- Consistent log format across modules

### Singleton Pattern
- Use singleton pattern for chat server: `global.m_chat_server_singelton_get_instance()`
- Ensure thread-safe initialization
- Document singleton usage

### Global Objects
- Several modules attached to `global` for easy access
- Be aware of global state
- Document global dependencies

### Memory Management
- Memory monitoring with auto-restart on limit exceeded
- Prevent memory leaks from causing crashes
- Monitor memory usage every 60 seconds

### Error Handling
- Check `global.m_logger` existence before use
- Implement comprehensive error handling
- Provide meaningful error messages

### Utilities
- Reuse utilities in `helpers/` directory
- Avoid code duplication
- Follow existing patterns

## Message Routing

Messages routed based on `ty` (type) and `tg` (target) fields:

### Message Types
- **'g'** (group broadcast) - Send to all members of a group
- **'i'** (individual) - Send to specific unit ID
- **'s'** (system/local) - System-level messages

### Message Targets
- `'_GCS_'` - All Ground Control Stations
- `'_GD_'` - All drone units
- `'_AGN_'` - All agents
- Specific unit ID - Individual targeting

### Loop Prevention
- Uses `_path` array to track message traversal
- Prevents infinite message loops
- Each server adds its ID to path

### Message Forwarding
- **Local Messages**: Forwarded to relay servers
- **External Messages**: Delivered locally only (no re-forwarding)

## S2S Authentication

### Ed25519 Cryptographic Signatures
- Child servers connect to parent with private key
- Parent servers verify child signatures with trusted public keys
- Challenge-response authentication flow
- Keys generated via authenticator's `scripts/gen_s2s_keys.sh`

### Authentication Flow
1. Child server initiates WebSocket connection
2. Parent server sends Ed25519 challenge
3. Child server signs challenge with private key
4. Parent server verifies signature with trusted public key
5. Connection established if signature valid
6. Persistent connection maintained

## Database

MySQL database for persistent storage:

### Tables
- User accounts
- Communication server registration
- Message history (if configured)
- Task persistence

### Operations
- Connection pooling
- Prepared statements
- Error handling
- Transaction support

## Development

### Setup
```bash
npm install
cp server.config server.config.local
# Edit server.config.local
mkdir -p server/ssl
openssl req -x509 -newkey rsa:4096 -keyout server/ssl/domain.key -out server/ssl/domain.crt -days 365 -nodes
npm start
```

### Testing
```bash
npm test
npm run test:watch
node --test test/unit/relay.test.js
```

## Deployment

### Parent Server
```bash
./deployment/run_parent.sh
```

### Child Server
```bash
./deployment/run_slave.sh
```

## Security Features

- SSL/TLS for WebSocket connections
- Ed25519 cryptographic S2S authentication
- Configurable trusted server keys
- Memory limit monitoring with auto-restart
- UDP proxy with kernel buffer management
- Temporary login key validation
- Account room isolation

## UDP Proxy

### Functionality
- Handles UDP packet forwarding between units
- Kernel buffer size checking and adjustment
- Fixed port configuration option
- Used for MAVLink and other UDP protocols

### Configuration
- Port configuration
- Buffer size limits
- Kernel parameter tuning

## Memory Management

### Monitoring
- Configurable memory limit (memory_max in MB)
- Automatic server restart when limit exceeded
- Memory monitoring every 60 seconds
- Prevents memory leaks from causing crashes

### Implementation
- Memory usage tracking
- Graceful shutdown on restart
- State preservation where possible

## Related Projects

- `droneegnage_authenticator` - Authentication server
- `droneengage_communication` - Communication protocol
- `droneengage_webclient_react` - React web client
- `droneengage_mavlink` - MAVLink integration

## Documentation

- `README.md` - User-facing documentation
- `wiki/MessagePropagation.md` - Message routing and relay architecture
- `wiki/S2SAuthentication.md` - Server-to-server authentication setup

## Version

3.9.11
