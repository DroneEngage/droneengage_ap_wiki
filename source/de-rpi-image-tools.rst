.. _de-rpi-image-tools:


========================
RPI Administration Tools
========================

The DroneEngage RPI image includes a web-based administration interface for easy configuration and management of your unit.

.. youtube::  -mV9EARM0_0

|

Accessing the Admin Interface
=============================

**When connected to the RPI Access Point:**

URL: ``https://192.168.9.1:9090``

**When RPI is on your WiFi network:**

1. Find the IP address: ``ping droneengage.local``
2. Access: ``https://<RPI_IP>:9090``

.. important::
   The admin interface uses HTTPS. Your browser may show a security warning - this is normal for self-signed certificates.

|

Available Tools
===============

.. toctree::
   :titlesonly:
   :maxdepth: 1
   
   WiFi Manager </de-rpi-image-tools-wifi>
   Account Setup </de-rpi-image-tools-account>
   App Management </de-rpi-image-tools-bash>
   Module Updater </de-rpi-image-module-updater>
   File Navigator </de-rpi-image-tools-navigator>
   Terminal </de-rpi-image-tools-terminal>

|

Quick Reference
===============

.. list-table::
   :widths: 30 70
   :header-rows: 1

   * - Tool
     - Purpose
   * - **WiFi Manager**
     - Configure network connectivity and Access Point
   * - **Account Setup**
     - Enter DroneEngage credentials and server settings
   * - **App Management**
     - Start/stop services, enable autostart, run simulator
   * - **Module Updater**
     - Update DroneEngage modules to latest versions
   * - **File Navigator**
     - Browse and edit configuration files
   * - **Terminal**
     - Run Linux commands directly on the RPI
   



