.. _webclient-whatis:


==================
WebClient
==================

Web-Client is website where you can track and control your drone from it.


**Web-Client URL**: `https://cloud.ardupilot.org:8001/webclient.html <https://cloud.ardupilot.org:8001/webclient.html>`_

**Source Code:** `https://github.com/DroneEngage/droneengage_webclient_react <https://github.com/DroneEngage/droneengage_webclient_react>`_  


.. youtube:: To3sngcdvh4&t=7s

|

Main Features
=============

#. Ability to control multiple drones at the same time.

#. Ability to stream videos from multiple drones simultaneously.

#. Ability to take photos with zoom based on mobile capabilities.

#. Ability to connect GamePad directly to web and fly drone smoothly.

#. Ability to connect to QGroundControl or Mission Planner using :ref:`webclient-udp-telemetry`

#. A phone-optimized **Mobile GCS view** at `cloud.ardupilot.org:8001/mobile <https://cloud.ardupilot.org:8001/mobile>`_ — no install required. Gives you a status bar with unit/signal info, an FPV viewer with fit/rotate/one-tap image save, and camera capture controls, sized for a phone screen. See :ref:`webclient-mobile`.

|

Getting Started
================

#. Navigate to the `Web Client <https://cloud.ardupilot.org:8001/webclient.html>`_.
#. Log in with your account email and access code.
#. Your connected units (Andruav or DroneEngage) will appear in the interface.
#. Click on a unit to view its status, video feed, and controls.

.. tip::
   For the best experience, use a modern browser (Chrome, Firefox, Edge) with hardware acceleration enabled.

|

For Developers
================

- **Repository**: `droneengage_webclient_react <https://github.com/DroneEngage/droneengage_webclient_react>`_
- **Stack**: React 19, Bootstrap 5, Leaflet maps, WebSocket, WebRTC.
- **Key files**: ``src/index.js``, ``src/js/js_andruav_auth.js``, ``src/js/server_comm/js_andruav_ws.js``, ``src/js/js_leafletmap.js``.
- **Configuration**: ``public/config.json`` controls endpoints, feature flags, and map providers.
- **WebConnector**: the ``webconnector/`` directory contains the local deployment helper (WebPlugin / LAN mode, for private-network deployments without the cloud server).
- **Runs in two modes**: Cloud mode (through the shared cloud server) or WebPlugin/LAN mode (local deployment).

See :doc:`technicals/webclient/de-web-technicals` for deeper architecture and protocol details.
