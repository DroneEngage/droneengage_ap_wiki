.. _de-glossary:


=====================
DroneEngage Glossary
=====================

This glossary defines key terms used throughout the DroneEngage documentation.

|

.. glossary::
    :sorted:

    4G/LTE
        Fourth Generation Long-Term Evolution cellular network technology. DroneEngage uses 4G/LTE modems to provide internet connectivity for unlimited range telemetry.

    Access Code
        A system-generated password used with your email to authenticate devices on the DroneEngage server. Obtain yours from `cloud.ardupilot.org <https://cloud.ardupilot.org>`_. Sharing your Access Code allows others to join your account (useful for shared GCS operations).

    Andruav
        `Andruav <https://play.google.com/store/apps/details?id=arudpilot.andruav&hl=en&gl=US>`_ is the Android-based predecessor to DroneEngage. It runs on Android phones mounted on drones. DroneEngage is the Linux-based evolution designed for Raspberry Pi.

    Ardupilot
        Open-source autopilot software that runs on :term:`FCB` hardware. Supports Copter, Plane, Rover, and Sub vehicles. See `ardupilot.org <https://ardupilot.org/>`_.

    Camera Module
        The DroneEngage module (``de_camera``) responsible for video streaming and photo capture. Supports Raspberry Pi cameras and USB cameras.

    Comm Module
        The main DroneEngage module (``de_comm``) that connects to the cloud server over the internet and manages communication between all local modules via the :term:`DataBus`.

    Companion Computer
        A secondary computer (typically Raspberry Pi) connected to the :term:`FCB` that provides additional capabilities like video streaming, 4G telemetry, and advanced processing.

    DataBus
        The internal UDP-based communication protocol used for inter-module communication within DroneEngage. Allows modules to exchange data efficiently.

    de_comm
        See :term:`Comm Module`.

    de_mavlink
        See :term:`MAVLink Module`.

    de_camera
        See :term:`Camera Module`.

    FCB
        Flight Control Board. Hardware that runs autopilot firmware (e.g., `Pixhawk <https://pixhawk.org/>`_, Cube, Kakute). Connects to the companion computer via serial or UDP.

    GCS
        Ground Control Station. Software used to monitor and control drones. Examples include `Mission Planner <https://ardupilot.org/planner/>`_ and `QGroundControl <http://qgroundcontrol.com/>`_.

    Geo-Fence
        A virtual boundary that restricts drone flight to specific areas. DroneEngage provides independent geo-fencing that works alongside or instead of the FCB's built-in geo-fence.

    JSON
        JavaScript Object Notation. A lightweight data format used for DroneEngage configuration files and communication protocols.

    MAVLink
        Micro Air Vehicle Link. A lightweight messaging protocol for communicating with drones. Used between the :term:`FCB` and :term:`MAVLink Module`.

    MAVLink Module
        The DroneEngage module (``de_mavlink``) that communicates directly with the :term:`FCB` via serial port or UDP. Translates between MAVLink and DroneEngage protocols.

    RPI
        Raspberry Pi. A single-board computer commonly used as a :term:`Companion Computer` for DroneEngage.

    RPI Image
        A pre-configured Raspberry Pi OS image with DroneEngage modules pre-installed. Available for RPI-Zero 2 W and RPI-4.

    SITL
        Software-In-The-Loop. A simulation mode that runs Ardupilot on a PC without physical hardware, useful for testing DroneEngage before flying.

    Smart Telemetry
        DroneEngage's bandwidth optimization feature that reduces data transmission on slower networks by filtering less critical MAVLink messages.

    Swarm
        A group of drones operating together under coordinated control. DroneEngage supports hierarchical swarm formations with leader and follower roles.

    TX Freeze
        A DroneEngage feature that locks the current throttle/control positions, useful for maintaining stable flight when the drone is beyond TX range.

    RC Blocking
        A safety feature allowing a field pilot with a physical transmitter to override all remote commands, providing emergency local control.

    Unit
        A DroneEngage-equipped drone or vehicle. Each unit connects to the cloud server and appears in the web client.

    Web Client
        The browser-based ground control station for DroneEngage. Access at `droneengage.com:8021/webclient.html <https://droneengage.com:8021/webclient.html>`_.
