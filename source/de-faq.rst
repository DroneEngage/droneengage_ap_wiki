.. _de-faq:

DroneEngage FAQ
===============

Find answers to common questions about using DroneEngage with Ardupilot. For additional terms, see :ref:`de-glossary`. Submit new questions via `GitHub Issues <https://github.com/DroneEngage/droneengage_communication/issues>`_.

1. **I don’t understand many terms used in this wiki.**

   - Check the :ref:`de-glossary` for definitions of terms like FCB (Flight Control Board), MAVLink, and JSON.
   - For a beginner’s overview, see :ref:`de-what-is` or :ref:`de-dev-building-code`.



2. **I can’t find my Access Code.**

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

   - Yes, new `RPI-Images <https://cloud.ardupilot.org/downloads/RPI_Full_Images/>`_ runs on RPI-5 as well as RPI-4 & RPIZero W2, as pre-built binaries are for RPI-WZero2 and RPI-4 (Bullseye 64-bit).
   - Follow :ref:`de-dev-building-code` for compilation steps.
   - For camera module support, additional changes are needed (see :ref:`de-config-camera`).


5. **How do I update to the latest DroneEngage binaries?**

   - Download the latest binaries from `https://cloud.ardupilot.org/downloads/RPI/Latest/ <https://cloud.ardupilot.org/downloads/RPI/Latest/>`_ (e.g., Drone_Engage_24Jul_2025.zip for telemetry, Drone_Engage_Camera_24Jul_2025.zip for camera).
   - Follow :ref:`de-install-unit` for installation steps.
   - Always verify the latest versions, as they update periodically.-
   - `READY-IMAGES <https://cloud.ardupilot.org/downloads/RPI_Full_Images/droneengage_rpi/>`_ setups are always more consistent and easier to install. 


6. **Why is my MAVLink connection failing?**

   - Ensure the correct `fcb_connection_uri` settings in :ref:`de-config-mavlink` (e.g., UDP or serial port).
   - Check that TX/RX pins are correctly connected between Raspberry Pi and :term:`FCB`.
   - Verify the baudrate matches your Flight Control Board settings.
   - If using dynamic port detection, ensure `dynamic: true` is set in the JSON config.


7. **How do I use the RPI Image Tools?**

   - Use the Navigator (:ref:`de-rpi-image-tools-navigator`) to manage files.
   - Use the Terminal (:ref:`de-rpi-image-tools-terminal`) for advanced CLI commands.


8. **What is Smart Telemetry, and how do I adjust it?**

   - Smart Telemetry reduces bandwidth by filtering non-critical data, ideal for slower networks.
   - Adjust levels (0–3) in `de_mavlink.config.module.json` (see :ref:`de-config-mavlink`).
     - Level 0: No optimization (full data).
     - Level 3: Maximum optimization (minimal data, slower GCS updates).


9. **Can I test DroneEngage without a drone?**

   - Yes, use Software-in-the-Loop (SITL) to simulate a drone on your computer.
   - See :ref:`de-simulators` for setup instructions.


10. **How do I contribute to the wiki or report issues?**

    - Edit content by submitting pull requests to `https://github.com/DroneEngage/droneengage_ap_wiki <https://github.com/DroneEngage/droneengage_ap_wiki>`_ (see :ref:`de-contributing`).
    - Report issues or suggest FAQs via GitHub Issues.

