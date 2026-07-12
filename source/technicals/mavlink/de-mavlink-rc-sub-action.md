# RC_SUB_ACTION Enumeration

`RC_SUB_ACTION` is an enumeration that defines the operational state of remote control (RC) channel input handling in a drone control system.  
It determines how RC commands from a transmitter (TX) are processed and whether they are passed through, modified, or overridden by software.

This enum is used to manage the autonomy level of RC input—ranging from fully released (no input sent) to active joystick control (direct channel override)—and plays a key role in safety and automation logic, especially during mode transitions like switching to guided flight.

---

## Definition

```cpp
typedef enum RC_SUB_ACTION 
{
    // No RC channel data is sent to the drone.
    RC_SUB_ACTION_RELEASED                      =   0,
    
    // Center values (1500 µs) are sent; physical TX has no effect.
    RC_SUB_ACTION_CENTER_CHANNELS               =   1,
    
    // Last known RC values are frozen and continuously sent.
    RC_SUB_ACTION_FREEZE_CHANNELS               =   2,
    
    // Live RC channel values from software (e.g. app) are sent.
    RC_SUB_ACTION_JOYSTICK_CHANNELS             =   4,
    
    // Velocity commands sent instead of raw RC; used in guided mode.
    RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED      =   8
} RC_SUB_ACTION;
```

- **Type**: `enum` (C++ scoped enumeration)
- **Purpose**: Controls how RC input is applied to the drone's flight controller
- **Used in**: `ANDRUAV_VEHICLE_INFO::rc_sub_action` to reflect current RC override state
- **Values are powers of two or sequential integers**, suggesting possible bitmask use or simple state tracking
- **Initial value**: Set to `RC_SUB_ACTION_RELEASED` during initialization

The enum is tightly coupled with RC override logic, where physical transmitter input can be disabled in favor of software-generated commands—critical for autonomous or app-controlled flight.

---

## Enum Values

| Value | Constant | Description |
|-------|----------|-------------|
| 0 | `RC_SUB_ACTION_RELEASED` | No RC channel data is sent to the drone |
| 1 | `RC_SUB_ACTION_CENTER_CHANNELS` | Center values (1500 µs) are sent; physical TX has no effect |
| 2 | `RC_SUB_ACTION_FREEZE_CHANNELS` | Last known RC values are frozen and continuously sent |
| 4 | `RC_SUB_ACTION_JOYSTICK_CHANNELS` | Live RC channel values from software (e.g. app) are sent |
| 8 | `RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED` | Velocity commands sent instead of raw RC; used in guided mode |

---

## Example Usages

### Initialization
One primary use is initializing the RC state when the system starts:

```cpp
de::fcb::mission::CMissionManager::getInstance().getAndruavMission().clear();

// Initialize RC override state to released (no input sent)
m_andruav_vehicle_info.rc_sub_action = RC_SUB_ACTION::RC_SUB_ACTION_RELEASED;
```

### Command Parsing
Another key usage is in parsing incoming commands from a remote source (e.g. mobile app):

```cpp
int rc_sub_action = cmd["b"].get<int>();

// Interpret integer command as RC_SUB_ACTION enum
m_fcbMain.adjustRemoteJoystickByMode((RC_SUB_ACTION)rc_sub_action);
```

Here, the JSON field `"b"` carries the desired RC mode, which is cast directly into the `RC_SUB_ACTION` enum and dispatched to control logic.

### Guided Mode Activation
The `RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED` mode is activated when the drone enters guided flight and joystick control is desired:

```cpp
m_andruav_vehicle_info.rc_sub_action =
    RC_SUB_ACTION::RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED;

m_fcb_facade.sendErrorMessage(..., std::string("RX Joystick Guided Mode"));
```

### Control Dispatch
In the control dispatch logic, both joystick modes are grouped but behavior diverges based on current flight mode:

```cpp
case RC_SUB_ACTION::RC_SUB_ACTION_JOYSTICK_CHANNELS:
case RC_SUB_ACTION::RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED: {
  if (m_andruav_vehicle_info.flying_mode == VEHICLE_MODE_GUIDED) {
    enableRemoteControlGuided(); // Uses ctrlGuidedVelocityInLocalFrame
  } else {
    enableRemoteControl(); // Uses sendRCChannels()
  }
}
```

### Timeout Handling
In the main control loop, timeout handling is applied for safety:

```cpp
case RC_SUB_ACTION::RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED: {
  if (now - m_andruav_vehicle_info.rc_command_last_update_time > RCCHANNEL_OVERRIDES_TIMEOUT) {
    releaseRemoteControl();
  }
}
```

This ensures safety: if no new joystick input arrives within `RCCHANNEL_OVERRIDES_TIMEOUT` (3 seconds), control is released and the drone may enter a safe mode (e.g., brake).

---

## Notes

- `RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED` is specifically intended for **guided mode flight**, where velocity commands (not raw PWM) are sent—this enables smoother autonomous maneuvers.
- The enum values skip `3` and `5–7`, suggesting possible future extensions or alignment with bitmask-style flags (though not currently used as bitflags).
- Despite being called "joystick", this system does **not require a physical joystick**—it refers to software-emulated RC inputs from apps or tracking systems.
- The system may **automatically switch** to `RC_SUB_ACTION_JOYSTICK_CHANNELS_GUIDED` from `RC_SUB_ACTION_JOYSTICK_CHANNELS` when entering guided flight.

---

## See Also

- `ANDRUAV_VEHICLE_INFO`: Struct that holds `rc_sub_action` as part of the drone's state; used for telemetry and control decisions.
- `CFCBMain::adjustRemoteJoystickByMode()`: Method that acts on `RC_SUB_ACTION` values to change RC behavior.
- `RCCHANNEL_OVERRIDES_TIMEOUT`: A related define (`3000000` µs) that governs how long overrides remain active before timing out.
- `mavlink::sendRCChannels()`: Function called when sending RC data in `CENTER`, `FREEZE`, or `JOYSTICK` modes.
- `ctrlGuidedVelocityInLocalFrame`: The actual control function used when `JOYSTICK_CHANNELS_GUIDED` mode is active; sends velocity commands to the drone.
