.. _de-dev:

===============
Developer Guide
===============

This section provides technical documentation for developers who want to build DroneEngage from source, create custom plugins, or understand the internal architecture.

Overview
========

DroneEngage is built with a modular architecture:

- **de_comm** - Communication module (connects to cloud server)
- **de_mavlink** - MAVLink module (connects to flight controller)
- **de_camera** - Camera module (video streaming and recording)
- **Plugins** - Custom modules for GPIO, SDR, tracking, etc.

All modules communicate via the **DataBus** using UDP sockets, allowing them to run on the same device or distributed across multiple devices.

Building & Setup
================

- `Building from Source <de-dev-building-code.html>`_

Raspberry Pi Deployment
=======================

- `Raspberry Pi Bookworm Scripts <technicals/rpi-scripts/rpi-bookworm-scripts.html>`_ - Overview of helper scripts for managing DroneEngage services, networking, cameras, simulators, and maintenance on Raspberry Pi OS

.. note::
   Camera Manager Wrapper: C++ wrapper for managing DroneEngage camera and tracking modules on Raspberry Pi. 
   See `GitHub <https://github.com/DroneEngage/DroneEngage_ScriptWiki/blob/master/rpi_image_scripts/bookworm/wrapper/README.html>`_



DroneEngage Internals
=====================

- `Architecture <de-dev-architecture.html>`_
- `Web Client Technicals <de-dev-webclient.html>`_

- `Communication Module Technicals <de-dev-communication.html>`_
- `Server Technicals <de-dev-server.html>`_
- `MAVLink Module Technicals <de-dev-mavlink.html>`_


Creating Custom Plugins
=======================

- `Custom Plugin Development <technicals/de_common/de-custom-plugins.html>`_
- `Broker Module <technicals/de_common/de-dev-plugins.html>`_


Source Code Repositories
========================

- `droneengage_communication <https://github.com/DroneEngage/droneengage_communication>`_ - Main communication module
- `droneengage_mavlink <https://github.com/DroneEngage/droneengage_mavlink>`_ - MAVLink interface
- `droneengage_camera <https://github.com/DroneEngage/droneengage_camera>`_ - Camera streaming
- `droneengage_databus <https://github.com/DroneEngage/droneengage_databus>`_ - Plugin development library
- `droneengage_webclient_react <https://github.com/DroneEngage/droneengage_webclient_react>`_ - Web client
