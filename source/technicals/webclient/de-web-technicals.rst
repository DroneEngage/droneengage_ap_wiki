Web Client Technicals
=====================

This section provides detailed technical documentation for the DroneEngage Web Client, covering the core components, architecture, and implementation details of the React-based web interface.

The web client serves as the primary Ground Control Station (GCS) interface for DroneEngage, providing real-time drone control, mission planning, video streaming, and system monitoring capabilities.

.. toctree::
   :titlesonly:
   :maxdepth: 1

   Authentication System <de-web-technicals-authentication>
   Configuration Management <de-web-technicals-configuration>
   Server Communication <de-web-technicals-servercomm>
   Site Configuration <de-web-technicals-siteconfig>
   Unit Management <de-web-technicals-unit>
   WebRTC Video Streaming <de-web-technicals-webrtc>
   Mission Planning Base <de-web-technicals-moduleplanning>

Core Components
---------------

- :doc:`Authentication System <de-web-technicals-authentication>`
- :doc:`Configuration Management <de-web-technicals-configuration>`
- :doc:`Server Communication <de-web-technicals-servercomm>`
- :doc:`Site Configuration <de-web-technicals-siteconfig>`
- :doc:`Unit Management <de-web-technicals-unit>`
- :doc:`WebRTC Video Streaming <de-web-technicals-webrtc>`
- :doc:`Mission Planning Base <de-web-technicals-moduleplanning>`

Overview
--------

The DroneEngage Web Client is built with React and provides a comprehensive interface for drone operations. Key architectural components include:

- **Authentication Layer** (\ ``CAndruavAuth``\ ) - Handles user authentication and session management
- **Communication Layer** (\ ``server_comm``\ ) - WebSocket-based real-time communication with the CommServer
- **Configuration System** (\ ``js_siteConfig``, ``js_globals``\ ) - Manages deployment and runtime settings
- **Unit Management** (\ ``CAndruavUnitObject``\ ) - Represents drones and GCS units in the system
- **Video Streaming** (\ ``js_webrtcstream``\ ) - WebRTC-based real-time video communication
- **Mission Planning** (\ ``ClssModulePlanningBase``\ ) - Dynamic form system for mission configuration

Architecture Principles
-----------------------

- **Singleton Pattern**: Core services use singleton instances for consistent state management
- **Event-Driven**: Components communicate via a global event emitter (\ ``js_eventEmitter``\ )
- **Modular Design**: Features are organized into independent modules with clear interfaces
- **Template-Based UI**: Dynamic forms use JSON templates for flexible configuration
- **Real-Time Communication**: WebSocket and WebRTC provide low-latency data and video streams

Key Files and Directories
-------------------------

- ``src/js/js_andruav_auth.js`` - Authentication system
- ``src/js/server_comm/`` - Communication layer modules
- ``src/js/js_siteConfig.js`` - Site configuration
- ``src/js/js_globals.js`` - Runtime configuration
- ``src/js/js_andruavUnit.js`` - Unit management
- ``src/js/js_webrtcthin2.js`` - WebRTC streaming
- ``src/components/planning/modules/`` - Mission planning components
- ``public/config.json`` - Deployment configuration overrides
