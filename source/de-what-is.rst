.. _de-what-is:

======================
What is DroneEngage?
======================

DroneEngage is a professional, cloud-based companion computer ecosystem designed for ArduPilot and PX4-based vehicles. It leverages cellular networks (4G/LTE/5G) and the internet to provide unlimited-range telemetry, high-definition video streaming, and advanced fleet management.

As the Linux-based evolution of the Andruav project, DroneEngage moves beyond mobile-phone-based hardware to support dedicated single-board computers like the Raspberry Pi, offering a modular and robust architecture for enterprise and long-range drone operations.

.. youtube:: Q-OyRnisq8U

|

Key Features
============

**Telemetry & Control**

- Unlimited range telemetry connection via 4G/LTE/5G networks
- Forward telemetry to/from ground stations (Mission Planner, QGroundControl)
- Remote control via gamepad over the internet
- Web-based ground control station accessible from anywhere

**Video & Camera**

- Stream video from one or more cameras
- Record video and capture images on the drone or remotely
- Support for Raspberry Pi cameras and USB cameras

**Fleet Management**

- Control multiple drones from a single interface
- Advanced geo-fencing independent of flight controller
- Swarm operations with synchronized missions
- Hierarchical swarm formations

**Safety Features**

- RC Blocking - field pilot can override remote control
- TX Freeze - maintain throttle position for long-range flights
- Independent geo-fence enforcement

|

In a Nutshell
=============

DroneEngage consists of:

- **Companion Software** - Runs on Raspberry Pi connected to your flight controller
- **Web Client** - Browser-based ground station for control and monitoring
- **Camera Module** - Video streaming and recording platform
- **Cloud Server** - Connects all components securely over the internet

|

Requirements
============

**Essential:**

- Raspberry Pi (Zero W, Zero 2 W, 3, 4, or 5)
- Radxa Board (Rock Pi 5, etc.) for more powerful processing.
- 4G/LTE USB modem or HAT for internet connectivity
- Ardupilot or PX4 flight controller
- DroneEngage account and Access Code from `cloud.ardupilot.org <https://cloud.ardupilot.org>`_

**Optional:**

- Camera(s) connected to the Raspberry Pi
- Xbox/PlayStation controller for remote piloting
- Sony IMX500 RPI-AI for DroneEngage AI capabilities
- Additional sensors or payloads (SDR, Light LEDS, Loud Speakers ...etc.)

|

.. note::
   DroneEngage is the Linux-based successor to `Andruav <https://cloud.ardupilot.org/downloads/Android/>`_, the Android companion app.

