.. _de-faq:

===============
DroneEngage FAQ
===============

Find answers to common questions about using DroneEngage with Ardupilot. For additional terms, see :ref:`de-glossary`. Submit new questions via `GitHub Issues <https://github.com/DroneEngage/droneengage_communication/issues>`_.

|

General Questions
=================

1. **I don't understand many terms used in this wiki.**

   - Check the :ref:`de-glossary` for definitions of terms like FCB (Flight Control Board), MAVLink, and JSON.
   - For a beginner's overview, see :ref:`de-what-is` or :ref:`de-dev-building-code`.

2. **I can't find my Access Code.**

   - Your :term:`Access Code` is sent to your registered email after creating an account (see :ref:`de-account-create`).
   - Check your spam or junk folder, as it may be filtered.
   - If still missing, create a new one or contact support via `mohammad.hefny@droneengage.com <mailto:mohammad.hefny@droneengage.com>`_.

3. **How many drones and Ground Control Stations (GCS) can connect simultaneously?**

   - Theoretically unlimited, but depends on:
     - Network quality (e.g., WiFi or GSM-Modem stability).
     - Data type (e.g., telemetry, video streaming).
     - Hardware performance (e.g., Raspberry Pi model).
   - See :ref:`de-telemetry` for optimizing bandwidth with Smart Telemetry.

4. **Does DroneEngage support Raspberry Pi 5?**

   - Yes, new `RPI-Images <https://cloud.ardupilot.org/downloads/RPI_Full_Images/>`_ runs on RPI-5 as well as RPI-4 & RPIZero W2.
   - Follow :ref:`de-dev-building-code` for compilation steps.

5. **Can I test DroneEngage without a drone?**

   - Yes! The RPI image includes a built-in simulator for 2 drones.
   - Use **Start Simulator** in the :ref:`de-rpi-image-module-updater` App Management screen.
   - Alternatively, use Software-in-the-Loop (SITL) on your computer - see :ref:`de-simulators`.

|

Installation & Updates
======================

6. **How do I update to the latest DroneEngage binaries?**

   - **Recommended:** Use the :ref:`de-rpi-image-module-updater` web interface to update modules automatically.
   - **Manual:** Download from `cloud.ardupilot.org/downloads/RPI/Latest/ <https://cloud.ardupilot.org/downloads/RPI/Latest/>`_.
   - `READY-IMAGES <https://cloud.ardupilot.org/downloads/RPI_Full_Images/droneengage_rpi/>`_ are always the easiest option.

7. **I updated a module and now it doesn't work. What happened?**

   - When modules are updated, they come with **fresh default configuration files**.
   - Your previous settings (account credentials, server URLs) are NOT preserved automatically.
   - **Solution:** Reconfigure the module via the WebAdmin interface, or restore specific fields from your backup.
   - See :ref:`de-rpi-image-module-updater` for detailed recovery steps.

8. **Where are my configuration backups stored?**

   - **Module backups** (automatic before each update): ``/home/pi/drone_engage_backups/``
   - **Config file backups** (manual via button): ``/home/pi/drone_engage_config_backups/``
   - Always click **Backup Configuration Files** before major updates.

9. **How do I roll back to a previous module version?**

   - Navigate to ``/home/pi/drone_engage_backups/``
   - Find the backup file (e.g., ``de_comm_20251203_172630.tar.gz``)
   - Extract it: ``tar -xzf ~/drone_engage_backups/de_comm_20251203_172630.tar.gz -C /home/pi/drone_engage/``
   - Restart DroneEngage services.

|

WiFi & Network
==============

10. **How do I connect my RPI to a WiFi network?**

    - Access the WebAdmin at ``https://192.168.9.1:9090`` (when connected to the ``DE_ADMIN`` AP).
    - Go to the WiFi Manager and enter your WiFi SSID and password.
    - Click **Configure WiFi** to connect.
    - See :ref:`de-rpi-image-tools-wifi` for details.

11. **I entered the wrong WiFi password and lost access to my RPI. How do I recover?**

    - Remove the SD card from the Raspberry Pi.
    - Insert it into a computer and create a file named ``activate_ap.txt`` in the **boot** folder.
    - Reinsert the SD card and reboot the RPI.
    - The ``DE_ADMIN`` Access Point will be enabled automatically.
    - Connect to it and reconfigure WiFi with the correct password.

12. **What are the default Access Point credentials?**

    - **SSID:** ``DE_ADMIN``
    - **Password:** ``droneengage``
    - Access the admin interface at ``https://192.168.9.1:9090``

13. **Can I use both WiFi and a 4G modem at the same time?**

    - Yes! The system can maintain internet via a GSM/4G modem independently of WiFi settings.
    - The AP mode is for local configuration access only and doesn't provide internet.
    - If a GSM modem is installed, the unit stays connected regardless of WiFi/AP mode.

|

Account & Configuration
=======================

14. **How do I set up my DroneEngage account on the RPI?**

    - Access the WebAdmin interface.
    - Go to **Account Setup** (:ref:`de-rpi-image-tools-account`).
    - Enter your Username and Access Code.
    - Click **Update Config** to save.

15. **What is an AirGap server and do I need one?**

    - An AirGap server is a private DroneEngage server for isolated networks.
    - Most users should leave the server as default (``cloud.ardupilot.org``).
    - Only change if you're running a private server installation.

16. **Which configuration files store my account credentials?**

    - ``/home/pi/drone_engage/de_comm/de_comm.config.module.json`` - Main module
    - ``/home/pi/simulator/sim_de_mavlink_instances/de_comm.1.config.module.json`` - Simulator 1
    - ``/home/pi/simulator/sim_de_mavlink_instances/de_comm.2.config.module.json`` - Simulator 2

|

Service Management
==================

17. **How do I start/stop DroneEngage services?**

    - **Via WebAdmin:** Use the App Management screen (:ref:`de-rpi-image-module-updater`).
    - **Via Terminal:**
    
      .. code-block:: bash

         sudo systemctl start de_comm    # Start
         sudo systemctl stop de_comm     # Stop
         sudo systemctl restart de_comm  # Restart

18. **How do I enable DroneEngage to start automatically on boot?**

    - **Via WebAdmin:** Click **Enable AutoStart DE** in App Management.
    - **Via Terminal:** ``sudo systemctl enable de_comm de_mavlink de_camera``

19. **What's the difference between "Stop DE" and "Disable AutoStart DE"?**

    - **Stop DE** - Temporarily stops services; they restart on next reboot.
    - **Disable AutoStart DE** - Permanently prevents services from starting until re-enabled.

20. **How do I apply configuration changes?**

    - Click **Clear DE Caches** to remove cached configurations.
    - Click **Restart DE** to restart services with new settings.
    - Or simply reboot the RPI.

|

Troubleshooting
===============

21. **Why is my MAVLink connection failing?**

    - Ensure correct ``fcb_connection_uri`` in :ref:`de-config-mavlink` (UDP or serial port).
    - Check TX/RX pins are correctly connected between RPI and :term:`FCB`.
    - Verify the baudrate matches your Flight Control Board settings.
    - If using dynamic port detection, ensure ``dynamic: true`` is set.

22. **How do I view service logs for debugging?**

    - **Via Terminal:**
    
      .. code-block:: bash

         sudo journalctl -u de_comm -n 50      # Last 50 lines
         sudo journalctl -u de_comm -f         # Follow live
         sudo journalctl -u de_mavlink -b      # Since last boot

    - Replace ``de_comm`` with ``de_mavlink`` or ``de_camera`` as needed.

23. **How do I check if DroneEngage services are running?**

    - **Via Terminal:** ``sudo systemctl status de_comm``
    - Look for ``active (running)`` in green.
    - Press ``q`` to exit the status view.

24. **My RPI is running slow. How do I check system resources?**

    - **Disk space:** ``df -h``
    - **Memory:** ``free -h``
    - **CPU temperature:** ``vcgencmd measure_temp``
    - **Running processes:** ``htop`` (press ``q`` to exit)

25. **How do I clear logs to free up disk space?**

    - Use **Clear Logs** in the App Management screen.
    - Or via terminal: ``sudo journalctl --vacuum-time=1d``

|

Telemetry & Video
=================

26. **What is Smart Telemetry, and how do I adjust it?**

    - Smart Telemetry reduces bandwidth by filtering non-critical data, ideal for slower networks.
    - Adjust levels (0–3) in ``de_mavlink.config.module.json`` (see :ref:`de-config-mavlink`).
      - Level 0: No optimization (full data).
      - Level 3: Maximum optimization (minimal data, slower GCS updates).

27. **How do I enable/disable the camera?**

    - **Via WebAdmin:** Use **Enable RPI-CAM** / **Disable RPI-CAM** in App Management.
    - **Via Terminal:** ``sudo systemctl start de_camera`` or ``sudo systemctl stop de_camera``

|

File Management
===============

28. **Where are DroneEngage modules installed?**

    - All modules are in ``/home/pi/drone_engage/``
    - List them with: ``ls ~/drone_engage``

29. **How do I access configuration files?**

    - Use the **Navigator** in WebAdmin (:ref:`de-rpi-image-tools-navigator`) for easy file browsing.
    - Or use the **Terminal** (:ref:`de-rpi-image-tools-terminal`) for direct access.
    - Config files are in each module folder (e.g., ``~/drone_engage/de_comm/de_comm.config.module.json``).

30. **How do I manually run a module for debugging?**

    - First stop the service: ``sudo systemctl stop de_comm``
    - Then run manually:
    
      .. code-block:: bash

         cd ~/drone_engage/de_comm
         sudo ./de_comm.so

|

Contributing
============

31. **How do I contribute to the wiki or report issues?**

    - Edit content by submitting pull requests to `github.com/DroneEngage/droneengage_ap_wiki <https://github.com/DroneEngage/droneengage_ap_wiki>`_ (see :ref:`de-contributing`).
    - Report issues or suggest FAQs via GitHub Issues.

