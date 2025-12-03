.. _de-telemetry:

=========
Telemetry
=========

DroneEngage provides unlimited range telemetry over 4G/LTE networks, enabling drone control from anywhere in the world.

.. youtube:: rZ-UkhF3WYw

|

.. toctree::
   :titlesonly:
   :maxdepth: 1

   UDP Telemetry Forwarding </webclient-udp-telemetry>

|

Connection Options
==================

DroneEngage connects to `Ardupilot-based flight controllers <https://ardupilot.org/copter/docs/common-autopilots.html>`_ via:

- **Serial** - Direct UART connection (recommended)
- **USB** - Via USB-to-serial adapter
- **UDP** - Network connection (for SITL or networked setups)
- **TCP** - Network connection
- **Bluetooth/WiFi** - Wireless local connections

|

Smart Telemetry
===============

DroneEngage includes bandwidth optimization (Smart Telemetry) that reduces data usage on slower networks:

- **Level 0** - No optimization (full data rate)
- **Level 1** - Light optimization
- **Level 2** - Moderate optimization
- **Level 3** - Maximum optimization (minimal bandwidth)

Configure in ``de_mavlink.config.module.json`` via the ``default_optimization_level`` field.

|

World Record Demo
=================

DroneEngage (formerly Andruav) demonstrated **12,193 km telemetry range** - controlling a car in Cairo, Egypt from Los Angeles, USA.

**View from USA:**

.. youtube:: DmpX-D10GyQ

**View from Egypt:**

.. youtube:: Kh4NU3FaFeE

.. tip::
   For remote control, use the :ref:`webclient-gamepad` feature for the best experience.


