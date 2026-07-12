# DroneEngage User Guide

Welcome to the DroneEngage documentation. This guide is organized to help you find information quickly based on your needs.

## Quick Start

If you're new to DroneEngage, start here:

- [What is DroneEngage?](de-what-is.rst) - Overview of the system
- [Getting Started](de-getting-started.rst) - Quick start guide
- [Installation](de-install.rst) - Installation instructions
- [Account Creation](de-account-create.rst) - Creating your account

## User Guides

### Basic Operations
- [Ground Control Station (GCS)](andruav-gcs.rst) - Using the web interface
- [Telemetry](de-telemetry.rst) - Understanding telemetry data
- [Flight Modes](andruav-advanced.rst) - Available flight modes
- [Geo-fencing](de-geo-fencing.rst) - Setting up geofences

### Advanced Features
- [Swarm Operations](de-swarm.rst) - Multi-drone coordination
- [Follow-Me](andruav-fpv.rst) - Tracking mode
- [RC Control](de-tx-block.rst) - Remote control setup
- [Mission Planning](andruav-communication-protocol-messages.rst) - Creating missions

### Configuration
- [Communication Module Configuration](de-config-comm.md) - Configure de_comm
- [MAVLink Module Configuration](de-config-mavlink.md) - Configure de_mavlink
- [Camera Configuration](de-camera.md) - Camera setup

### Hardware Setup
- [Hardware Requirements](de-hw_2.rst) - Supported hardware
- [Unit Installation](de-install-unit.rst) - Installing on companion computers
- [Simulators](de-simulators.rst) - Using simulators for testing

## Technical Documentation

For detailed technical information, architecture, and development guides, see the [Technical Documentation](technicals/de-system-architecture.md).

### System Architecture
- [System Architecture Overview](technicals/de-system-architecture.md) - High-level system design
- [Authentication Server](technicals/server/de-server-technicals-authenticator.md) - Authentication system details
- [Communication Server](technicals/server/de-server-technicals-communication.md) - Communication server details
- [Communication Module](technicals/communication/de-comm-technicals.md) - C++ communication broker
- [MAVLink Module](technicals/mavlink/de-mavlink-technicals.md) - Flight controller interface

### Server Deployment
- [Server Installation](srv-Installation.rst) - Server setup guide
- [Authentication Server Setup](srv-authentication.rst) - Authentication server configuration
- [Communication Server Setup](srv-communication.rst) - Communication server configuration
- [Airgap Installation](srv-install-airgap.rst) - Offline installation

## Developer Documentation

For developers who want to extend or contribute to DroneEngage:

- [Developer Guide](de-dev.md) - Development overview
- [Building from Source](de-dev-building-code.rst) - Compilation instructions
- [Plugin Development](de-custom-plugins.md) - Creating custom plugins
  - [C++ Plugins](de-custom-plugins-cpp.md)
  - [Node.js Plugins](de-custom-plugins-nodejs.md)
  - [Python Plugins](de-custom-plugins-python.md)
- [DataBus](de-dev-databus.md) - Inter-module communication
- [Communication Protocol](de-dev-andruav-communication-protocol.md) - Protocol details

## Web Client Documentation

- [Web Client Overview](webclient-whatis.rst) - Web interface guide
- [Gamepad Control](webclient-gamepad.rst) - Using gamepad for control
- [Swarm Control](webclient-swarm.rst) - Managing swarms from web
- [UDP Telemetry](webclient-udp-telemetry.rst) - UDP telemetry setup

## FAQ and Troubleshooting

- [FAQ](de-faq.rst) - Frequently asked questions
- [Glossary](glossary.rst) - Terminology reference

## Use Cases

- [Use Cases](de-use-cases.rst) - Common usage scenarios
- [Scenarios](scenarios/) - Detailed scenario guides

## Contributing

- [Contributing Guide](de-contributing.rst) - How to contribute

## Documentation Structure

```
source/
├── de-*.rst/md           # User guides and basic documentation
├── andruav-*.rst         # Legacy Android app documentation
├── technicals/           # Technical documentation
│   ├── de-system-architecture.md
│   ├── server/           # Server technical docs
│   ├── communication/    # Communication module technical docs
│   ├── mavlink/          # MAVLink module technical docs
│   └── de_common/        # Common components technical docs
├── srv-*.rst             # Server deployment guides
├── de-dev-*.rst/md       # Developer documentation
├── webclient-*.rst       # Web client documentation
├── scenarios/            # Detailed scenario guides
└── use-cases/            # Use case examples
```

## Getting Help

If you can't find what you're looking for:
1. Check the [FAQ](de-faq.rst)
2. Search the [Glossary](glossary.rst) for terminology
3. Review [Technical Documentation](technicals/de-system-architecture.md) for architecture details
4. Check [Developer Documentation](de-dev.md) for implementation details
