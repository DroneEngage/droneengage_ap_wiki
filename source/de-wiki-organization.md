# DroneEngage Wiki Organization

This document describes the organization and structure of the DroneEngage documentation wiki.

## Documentation Hierarchy

The wiki is organized into four main sections:

### 1. User Guides
Easy-to-follow guides for users and operators.

- **Getting Started** (`de-getting-started.rst`)
  - Installation guides
  - Quick start tutorials
  - Basic configuration

- **User Guides** (`de-user-guide-index.md`)
  - Comprehensive user guide index
  - Feature guides
  - Operation manuals

- **Web Client** (`webclient-*.rst`)
  - Web client configuration
  - Gamepad control
  - Servo control
  - Swarm operations
  - UDP telemetry

- **Scenarios** (`scenarios/`)
  - All-in-one setup
  - Camera names
  - Drone connection
  - Use case examples

### 2. Technical Documentation
In-depth technical documentation for developers and system administrators.

#### Server Components (`technicals/server/`)
- **System Architecture** (`de-system-architecture.md`)
  - Overall system architecture
  - Component interactions
  - Data flow diagrams

- **Authenticator** (`de-server-technicals-authenticator.md`)
  - Tech stack and architecture
  - Core components
  - Configuration
  - Development guide

- **Communication Server** (`de-server-technicals-communication.md`)
  - Tech stack and architecture
  - Message routing
  - S2S authentication
  - UDP proxy

- **API Endpoints** (`de-server-api-endpoints.md`)
  - REST API reference
  - Admin endpoints
  - Agent endpoints
  - Web endpoints

- **Authentication Flow** (`de-server-authentication-flow.md`)
  - Login card creation
  - Account operations
  - Hardware verification
  - S2S authentication

- **Configuration** (`de-server-configuration.md`)
  - Server settings
  - Account storage
  - S2S configuration
  - SSL/TLS setup

- **Database Schema** (`de-server-database-schema.md`)
  - File-based storage
  - MySQL schema
  - Database operations

- **Message Propagation** (`de-server-message-propagation.md`)
  - Mesh relay system
  - Loop prevention
  - Message routing types

- **S2S Authentication** (`de-server-s2s-authentication.md`)
  - Ed25519 key setup
  - Challenge-response flow
  - Security best practices

#### Communication Module (`technicals/communication/`)
- **Technical Overview** (`de-comm-technicals.md`)
  - Tech stack and architecture
  - Core components
  - Build instructions

- **Plugin-Broker Architecture** (`de-comm-plugin-broker-architecture.md`)
  - Plugin side components
  - Broker side components
  - Communication flow
  - Thread architecture

- **Configuration System** (`de-comm-configuration-system.md`)
  - CConfigFile singleton
  - Configuration values
  - Usage patterns

#### MAVLink Module (`technicals/mavlink/`)
- **Technical Overview** (`de-mavlink-technicals.md`)
  - Tech stack and architecture
  - Key features
  - Vehicle types
  - Flight modes

- **Configuration System** (`de-mavlink-configuration-system.md`)
  - CConfigFile singleton
  - RC channel configuration
  - Tracking configuration
  - Network configuration

- **RC Sub-Action** (`de-mavlink-rc-sub-action.md`)
  - RC_SUB_ACTION enumeration
  - Remote control modes
  - Timeout handling

- **Rate Limit Effect** (`de-mavlink-rate-limit-effect.md`)
  - Tracking rate limiting
  - Kalman filter interaction
  - Configuration ranges

- **Joystick Guided Mode** (`de-mavlink-joystick-guided-mode.md`)
  - Guided mode control
  - Velocity setpoints
  - Safety features

### 3. API Documentation
Complete API references for all components.

- **REST APIs** (`technicals/server/de-server-api-endpoints.md`)
  - Authenticator API
  - Communication Server API
  - Error codes
  - Response formats

- **WebSocket APIs** (in component technical docs)
  - Message protocols
  - Connection handling
  - Event types

- **MAVLink APIs** (in MAVLink technical docs)
  - Message types
  - Vehicle commands
  - Telemetry formats

### 4. Architecture Documentation
System architecture and design documents.

- **System Architecture** (`technicals/de-system-architecture.md`)
  - High-level architecture
  - Component diagram
  - Data flow
  - Integration points

- **Communication Architecture** (`technicals/communication/de-comm-plugin-broker-architecture.md`)
  - Plugin-broker pattern
  - Message routing
  - UDP communication

- **Server Architecture** (in server technical docs)
  - Authenticator architecture
  - Communication Server architecture
  - S2S architecture

## Development Documentation

For developers contributing to DroneEngage:

- **Development Guide** (`de-dev.md`)
  - Building code
  - Testing
  - Contributing guidelines

- **Custom Plugins** (`de-custom-plugins.md`)
  - C++ plugins
  - Node.js plugins
  - Python plugins

- **Databus** (`de-dev-databus.md`)
  - Inter-module communication
  - Message protocol
  - Plugin development

## Server Installation

Server deployment and installation guides:

- **Server Index** (`srv-index.rst`)
- **Authentication** (`srv-authentication.rst`)
- **Communication** (`srv-communication.rst`)
- **Installation** (`srv-install-*.rst`)

## Legacy Documentation

Some documentation files are kept for backward compatibility:

- **Andruav Documentation** (`andruav-*.rst`)
  - Original Andruav system docs
  - Protocol messages
  - FAQ
  - Getting started

## Navigation

- **Main Index** (`index.rst`) - Main wiki entry point
- **DE Index** (`de-index.rst`) - DroneEngage specific index
- **Glossary** (`glossary.rst`) - Terminology and definitions

## File Naming Conventions

- User guides: `de-*.rst` or `de-*.md`
- Technical docs: `technicals/[component]/de-[component]-[topic].md`
- API docs: `de-server-api-endpoints.md`
- Architecture: `de-system-architecture.md`
- Server docs: `srv-*.rst`
- Web client docs: `webclient-*.rst`

## Updating Documentation

When adding new documentation:

1. Determine the appropriate section (User Guides, Technical, API, Architecture)
2. Follow the naming convention for that section
3. Update the relevant index file
4. Add cross-references to related documents
5. Ensure consistent formatting and structure
