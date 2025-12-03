.. _de-dev:

================
Developer Guide
================

This section provides technical documentation for developers who want to build DroneEngage from source, create custom plugins, or understand the internal architecture.

|

.. toctree::
   :caption: Building & Setup
   :titlesonly:
   :maxdepth: 1
   
   Building from Source </de-dev-building-code>

.. toctree::
   :caption: Architecture
   :titlesonly:
   :maxdepth: 1

   Communication Protocol </de-dev-andruav-communication-protocol>
   DataBus (Inter-Module Communication) </de-dev-databus>

.. toctree::
   :caption: Extending DroneEngage
   :titlesonly:
   :maxdepth: 1

   Creating Custom Plugins </de-dev-plugin>
   SWARM Logic </de-dev-swarm>
   Communication Module Config </de-config-comm>
   MAVLink Module Config </de-config-mavlink>
|

Overview
========

DroneEngage is built with a modular architecture:

- **de_comm** - Communication module (connects to cloud server)
- **de_mavlink** - MAVLink module (connects to flight controller)
- **de_camera** - Camera module (video streaming and recording)
- **Plugins** - Custom modules for GPIO, SDR, tracking, etc.

All modules communicate via the **DataBus** using UDP sockets, allowing them to run on the same device or distributed across multiple devices.

Source Code Repositories
========================

- `droneengage_communication <https://github.com/DroneEngage/droneengage_communication>`_ - Main communication module
- `droneengage_mavlink <https://github.com/DroneEngage/droneengage_mavlink>`_ - MAVLink interface
- `droneengage_camera <https://github.com/DroneEngage/droneengage_camera>`_ - Camera streaming
- `droneengage_databus <https://github.com/DroneEngage/droneengage_databus>`_ - Plugin development library
- `droneengage_webclient_react <https://github.com/DroneEngage/droneengage_webclient_react>`_ - Web client




