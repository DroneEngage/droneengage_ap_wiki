# Scenarios

This page covers various scenarios and use cases for DroneEngage, including configuration examples, real-world deployments, and solutions to common problems.


## On-Board DroneEngage
This is the [default DroneEngage installation](de-drone-connection.md), where DroneEngage runs on a Raspberry Pi or similar board installed directly on the drone. More about this configuration can be found [here](de-drone-connection.md).


## Ground-DroneEngage (Air Unit)
This is a [very suitable configuration](de-drone-ground-config-unit.md) for drones that already have telemetry and video links, allowing access via DroneEngage without modifying the onboard systems. It is also ideal when a powerful ground-based computer is needed for AI/ML processing. More about this configuration can be found [here](de-drone-ground-config-unit.md).



## All-In-One Board (RPI as FCB + Companion)
This configuration uses a single Raspberry Pi 4 board to run both the flight controller (Ardupilot) and the companion computer (DroneEngage), leveraging CPU affinity for real-time performance. More about this configuration can be found [here](de-all-in-one.html).


## Camera Naming & Video Pipelines
When multiple cameras are connected, the same camera may not always receive the same device path (e.g., `/dev/videoX`) after a board restart or when a new camera is connected. This variability can make debugging frustrating and time-consuming.

To address this, [DroneEngage creates virtual camera devices](de-camera-names.md) using <a href="https://github.com/umlaeute/v4l2loopback" target="_blank">v4l2loopback</a>, assigning custom names to each camera source for reliable identification.

Each time cameras are initialized, they may receive a different index (i.e., `/dev/videoX`, where `X` changes across restarts or new connections). DroneEngage eliminates this uncertainty by using custom names, allowing the Camera, Tracker, and AI modules to access cameras by name rather than by device path. More details can be found [here](de-camera-names.md).



