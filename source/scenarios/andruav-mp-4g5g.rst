.. _andruav-mp-4g5g:

=====================================================
Andruav + Mission Planner over 4G/5G, with Live Video
=====================================================

This is the classic Andruav setup: no radio telemetry module, no analog FPV
gear — just two Android phones and a cellular connection, giving you
Mission Planner control and live video anywhere there's 4G/5G or WiFi.

.. youtube:: DmpX-D10GyQ

|

.. tip::

   This is the same link that carried radio control input for a
   12,193 km demo — a car driven in Cairo, Egypt, with the driver's
   controller in Los Angeles, USA. See :ref:`andruav-telemetry`.

|

What You Need
==============

- An Android phone mounted on the drone/vehicle, running Andruav, connected
  to the ArduPilot flight controller (Bluetooth, USB/Serial, or UDP — see
  :ref:`andruav-serial`)
- A SIM with 4G/5G data (or WiFi) on the drone-side phone
- A free Andruav account and access code — see :ref:`andruav-getting-started`
- Mission Planner (or QGroundControl) on the ground, plus a browser for the
  WebClient

|

How It Connects
================

The drone-side phone is the only thing that needs a cellular connection.
Everything else — Mission Planner, the WebClient, a second phone — connects
to it through the cloud server, so none of them need to be reachable from
the internet themselves:

.. code-block:: text

   Flight Controller  --(BT/USB/UDP)-->  Android Phone (Andruav)
                                                |
                                          4G/5G/WiFi
                                                |
                                    Cloud.Ardupilot.org
                                          /          \
                          UDP Telemetry Proxy      WebRTC Video
                                    |                     |
                          Mission Planner /      WebClient / /mobile
                          QGroundControl (UDPCI)  (live FPV view)

|

Setup Steps
===========

1. **Link the flight controller.** On the drone phone, open the FCB
   connection sheet and pick Bluetooth, USB, or WiFi/UDP depending on how
   your board is wired.

   .. image:: ../images/new_andruav_2026/andruav_fcb_bluetooth.png
      :height: 480px
      :align: center
      :alt: Andruav FCB connection sheet with Bluetooth, USB, WiFi and UDP options

2. **Confirm the cloud connection.** The Comm Server screen should show
   ``Cloud.Ardupilot.org`` connected, with your Unit ID and access code
   set. This is the same link the WebClient and Mission Planner will both
   reach through.

   .. image:: ../images/new_andruav_2026/andruav_comm_server.png
      :height: 480px
      :align: center
      :alt: Andruav Comm Server settings screen

3. **Forward telemetry to Mission Planner.** Enable the UDP Proxy — from the
   phone's settings, or from the WebClient's Smart Telemetry panel (also
   reachable from the :ref:`webclient-mobile`) — and point Mission
   Planner's UDPCI connection at the IP/port it gives you. Full detail in
   :ref:`webclient-udp-telemetry`.

   .. image:: ../images/new_andruav_2026/webclient_mobile_andruav_telemetry.png
      :height: 480px
      :align: center
      :alt: Smart Telemetry panel on the mobile WebClient showing the UDP proxy IP and port for Mission Planner

4. **Start live video.** Open :ref:`webclient-whatis` (or the lightweight
   :ref:`webclient-mobile` on a second phone), sign in with the same
   access code, and press the video icon on the drone's card to start
   streaming.

   .. image:: ../images/new_andruav_2026/andruav_fpv.png
      :width: 90%
      :align: center
      :alt: Live FPV video with telemetry overlay received over 4G/5G

|

Result
======

Full Mission Planner control (arm, mode changes, mission upload, RC over
the same link) and a live HD video feed, both riding entirely on cellular
data — no line-of-sight limit, no separate FPV transmitter, and nothing
that needs a public IP or port-forwarding on your end.
