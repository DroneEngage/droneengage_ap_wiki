# Raspberry Pi Bookworm Scripts

This page provides an overview of the helper and management scripts for Raspberry Pi OS (Bookworm/Bullseye) used to operate DroneEngage services, networking, cameras, simulators, and maintenance tasks.

For the complete documentation, see the [Raspberry Pi Bookworm Scripts README](https://github.com/DroneEngage/DroneEngage_ScriptWiki/blob/master/rpi_image_scripts/bookworm/README.md).

## Subfolders

- **c_helpers/** - C++ helper utilities including `updateConfig` tool for updating DroneEngage JSON config files with username, access code, and server settings
- **service/** - Systemd service unit files for DroneEngage modules (de_communicator, de_mavlink, de_camera, etc.)
- **updates/** - Scripts for configuration backup and OTA updates
- **wrapper/** - Camera manager wrapper (C++) for orchestrating camera pipelines and tracking modules
- **not_used_but_useful/** - Archive of scripts not currently in use but potentially useful for reference

## Service Management Scripts

- **enable_and_restart_services.sh** - Enables and starts core DroneEngage services (de_communicator, de_mavlink)
- **disable_droneengage_service.sh** - Disables and stops DroneEngage services
- **restart_droneengage_services.sh** - Restarts DroneEngage services
- **stop_droneengage_services.sh** - Stops de_communicator and de_mavlink services
- **enable_and_restart_rpi_cam.sh** - Enables and starts de_camera_rpi_cam.service
- **stop_rpi_cam.sh** - Stops de_camera_rpi_cam.service

## Networking (Wi-Fi) Scripts

- **create_ap.sh** - Sets up a classic AP using hostapd and dnsmasq with static IP, DHCP range, and NAT
- **wifi_start_ap.sh** - Creates a NetworkManager hotspot AP (SSID `DE_ADMIN`, WPA2) on wlan0
- **wifi_use_wlan.sh** - Connects to a Wi-Fi network as a client using NetworkManager
- **wifi_clean_all_non_ap.sh** - Removes all NetworkManager connections except the hotspot AP

## Camera and Video Scripts

- **sh_camera_create_named_vc.sh** - Loads v4l2loopback to create named virtual cameras (DE-CAM1, DE-CAM2, DE-TRK, DE-RPI, DE-THERMAL)
- **sh_camera_run_rpi_camera.sh** - Streams from Raspberry Pi camera using rpicam-vid to DE-RPI virtual camera
- **sh_camera_senxor_thermal_run_on_vc.sh** - Runs thermal pipeline to DE-THERMAL virtual camera
- **sh_stream_from_camera.sh** - Simple GStreamer pipeline from libcamerasrc to v4l2 sink
- **sh_kill_all_camera_apps.sh** - Force-kills camera-related processes

These scripts are used by the [Camera Manager Wrapper](../../scenarios/de-camera-names.md)

## Simulator Scripts

- **sh_start_simulators.sh** - Starts de_comm, de_mavlink, and ArduPilot SITL instances
- **sh_stop_simulators.sh** - Gracefully terminates running simulator processes

## Configuration and Utility Scripts

- **sh_update_de_comm_config_in_sim.sh** - Updates simulation config JSON with user credentials
- **sh_reset_config_local_files.sh** - Deletes all *.local files under drone_engage directory
- **hlp_delete_file_instances.sh** - Searches and deletes all instances of a given filename
- **hlp_check_open_ports.sh** - Displays open TCP/UDP ports by PID or application name
- **hlp_reset_oem.sh** - Runs config update with placeholder credentials, starts AP, cleans logs
- **isBullseye.sh** - Detects whether OS is Raspberry Pi OS Bullseye or Bookworm

## Maintenance Scripts

- **sh_clean_logs.sh** - Aggressive cleanup for image sanitization: clears journals, logs, temp files, APT caches, old kernels, user caches, SSH keys, etc.

## Notes

- Many scripts require `sudo` and assume specific paths under `/home/pi`
- Camera scripts expect `v4l2loopback`, `ffmpeg`, and `rpicam-apps` to be installed
- Networking scripts may disrupt connectivity; run from console or be ready to reconnect
