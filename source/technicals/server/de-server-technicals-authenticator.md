# Authentication Server Technicals

## Overview

The DroneEngage Authentication Server (`droneegnage_authenticator`) is the central authentication and authorization server for the DroneEngage/Andruav drone management ecosystem. It handles user registration, vehicle registration, authentication token generation, and assigns users/vehicles to communication servers using Ed25519 cryptographic authentication for server-to-server communication.

## Tech Stack

- **Runtime**: Node.js >= 18
- **Web Framework**: Express.js
- **Database**: SQLite3 with migration support
- **Authentication**: Custom token-based + Ed25519 S2S authentication
- **Real-time**: WebSocket (ws library)
- **Templates**: EJS with express-ejs-layouts
- **Security**: Helmet, CORS, CSRF protection, express-rate-limit

## Architecture

Three independent servers running simultaneously:

### 1. API Server (port 19408)
REST API for authentication operations:
- User login/logout
- User registration
- Token verification
- Server-to-server registration

### 2. Views Server (port 8089)
Admin web interface with EJS templating:
- Dashboard for monitoring
- Server management
- User management
- Configuration interface

### 3. S2S WebSocket Server (port 19001)
Server-to-Server communication with Ed25519 authentication:
- Challenge-response authentication
- Signature verification
- Trusted key management
- Persistent connections to communication servers

## Key Files

### Core Server Files
- `server.js` - Main entry point, spawns all three servers
- `server.config` - JSON configuration (supports --config override)
- `package.json` - Dependencies and scripts
- `js_constants.js` - API endpoints, error codes, message types

### Authentication Server Components
- `auth_server/js_auth_server.js` - Main auth server controller
- `auth_server/js_account_manager.js` - User account CRUD operations
- `auth_server/js_comm_server_manager.js` - Communication server management
- `auth_server/js_session_manager.js` - Session/token management
- `auth_server/js_s2s_auth.js` - Server-to-Server Ed25519 authentication
- `auth_server/js_database_manager.js` - Database operations layer

### Routes
- `routes/js_router.js` - Main router setup
- `routes/js_router_admin.js` - Admin interface routes
- `routes/js_router_agent.js` - Agent/drone routes
- `routes/js_router_web.js` - Web client routes

### Database
- `database/db_users.js` - User database operations
- `database/migrations/` - SQL migration files

### Helpers
- `helpers/hlp_args.js` - Argument parsing utilities
- `helpers/hlp_db.js` - Database helper functions
- `helpers/hlp_string.js` - String manipulation utilities
- `helpers/hlp_validation.js` - Input validation

## Configuration

Configuration is centralized in `server.config` (JSON format). Key sections:

### Server Identity
- `server_id` - Unique server identifier
- `server_ip` - Server IP address
- `server_port` - Server port number

### Account Storage
- `storage_mode` - Storage backend (single/file/db)
- Account data persistence configuration

### S2S Authentication
- Trusted Ed25519 public keys
- Private key for signature generation
- Key rotation policies

### SSL/TLS
- Certificate paths
- SSL configuration options

### Admin Interface
- Admin credentials
- Interface configuration

### Logging
- Log level configuration
- Log file paths
- Log rotation settings

## Coding Standards

### Logging
- Use `global.m_logger` for logging (if enabled)
- Check `global.m_logger` existence before use
- Consistent log format across modules

### Input Validation
- All inputs must pass `auth_server/js_input_validator.js`
- Validate user input before processing
- Sanitize data to prevent injection attacks

### Database Operations
- Use `auth_server/js_database_manager.js` for all DB operations
- Use prepared statements to prevent SQL injection
- Handle database errors gracefully

### Token Management
- Use `auth_server/js_session_manager.js` for token operations
- Implement token expiration
- Secure token storage

### Utilities
- Reuse utilities in `helpers/` directory
- Avoid code duplication
- Follow existing patterns

### Global Objects
- Several modules attached to `global` for easy access
- Be aware of global state
- Document global dependencies

### Error Handling
- Check `global.m_logger` existence before use
- Implement comprehensive error handling
- Provide meaningful error messages

## Authentication Flow

### User Registration
1. User submits registration form (username, email, password)
2. Server validates input
3. Server generates access code
4. Server sends access code via email (unless `ignoreEmail` is set)
5. User verifies access code
6. Account activated

### User Login
1. User submits login credentials (username/email, password)
2. Server validates credentials
3. Server generates session token
4. Server returns token to client
5. Client uses token for subsequent requests

### S2S Authentication
1. Communication server initiates WebSocket connection
2. Authenticator sends Ed25519 challenge
3. Communication server signs challenge with private key
4. Authenticator verifies signature with trusted public key
5. Connection established if signature valid
6. Persistent connection maintained

## API Endpoints

### Authentication Endpoints
- `POST /api/login` - User login
- `POST /api/register` - User registration
- `POST /api/logout` - User logout
- `POST /api/verify` - Token verification

### S2S Endpoints
- `POST /api/s2s/register` - Register communication server
- `GET /api/s2s/servers` - List registered servers
- `WS /s2s` - WebSocket endpoint for S2S communication

### Admin Endpoints
- `GET /admin/dashboard` - Admin dashboard
- `GET /admin/servers` - Server management
- `GET /admin/users` - User management

### Agent Endpoints
- `POST /agent/register` - Register drone agent
- `POST /agent/authenticate` - Authenticate drone agent
- `GET /agent/servers` - Get assigned communication server

## Database Schema

SQLite database with tables:

### users table
- `id` - Primary key
- `username` - Unique username
- `email` - User email
- `password_hash` - Hashed password
- `access_code` - Verification code
- `created_at` - Account creation timestamp

### comm_servers table
- `id` - Primary key
- `server_id` - Unique server identifier
- `ip_address` - Server IP address
- `port` - Server port
- `public_key` - Ed25519 public key
- `registered_at` - Registration timestamp

## Security Features

### HTTP Security
- Helmet.js for HTTP security headers
- CORS configuration for cross-origin requests
- CSRF protection for form submissions
- Rate limiting on API endpoints

### Transport Security
- SSL/TLS for all connections
- Certificate validation
- Secure cipher suites

### Authentication Security
- Ed25519 cryptographic S2S authentication
- Secure password hashing (bcrypt)
- Configurable session secrets
- Token expiration

### Input Security
- Input validation on all endpoints
- SQL injection prevention
- XSS protection
- Command injection prevention

## Development

### Setup
```bash
npm install
cp server.config server.config.local
# Edit server.config.local
mkdir -p ssl
openssl req -x509 -newkey rsa:4096 -keyout ssl/domain.key -out ssl/domain.crt -days 365 -nodes
npm start
```

### Testing
```bash
npm test
node --test test/js_session_manager.test.js
```

## S2S Key Generation

Generate Ed25519 keys for server-to-server authentication:

```bash
cd scripts
./gen_s2s_keys.sh <server_id>
```

This generates:
- Private key for the server
- Public key to share with authenticator
- Key pair stored in secure location

## Related Projects

- `droneengage_communication` - Communication server
- `droneengage_server` - Main DroneEngage server
- `droneengage_webclient_react` - React web client
- `droneengage_mavlink` - MAVLink integration

## Documentation

- `README.md` - User-facing documentation
- `wiki/AuthenticationFlow.md` - Detailed authentication flow
- `wiki/DatabaseSchema.md` - Database schema documentation
- `wiki/APIEndpoints.md` - API endpoint reference
- `wiki/Configuration.md` - Configuration guide
- `wiki/S2SAuthentication.md` - Server-to-server authentication setup

## Version

5.0.0
