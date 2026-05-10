# Ground DroneEngage
This is suitable when you already have a drone with no room for another board, or where it already have its telemetry and video links and you need to connect it to DroneEngage.

In this solution, the FCB (Flight Control Board) is connected to the ground -DroneEngage unit that can run on a laptop computer or any other device.

DroneEngage unit itself is on the ground although the FCB is airborne. Cameras ara also on the drone and transmitted to the ground via video link. All you need to do is to provide a telemetry data to ground computer via serial port-usb converter or via tcp/udp connection which is supported by most data links.

For videos you need a video grapper that captures from HDMI output from video link.


You can use one computer for multiple units on air, the only condition is that you need to have enough ports and video grappers for each unit on the ground computer and enough processing power.


## Advantages Of This Configuration
- You can use a powerful groud computer to run compless Neural Networks and other heavy processing tasks.
- You dont need to modify your drone hardware.
- For very long range where you already have your own powerful telemetry and video links that are not TCP/IP.


## Disadvantages Of This Configuration
- You need a active reliable connection between the ground computer - drone brain- and the drone itself. you cannot fly independent offline missions.
- Incase of lost connection, the drone will rely on Ardupilot failsafe scenarios with no DroneEngage extra options.




This is actually a very nice scenario for testing your AI and tracking software, before deploying it on a drone body.