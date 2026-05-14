# Custom Plugin Development

Writing Custom Plugins allows you to create modules that handle hardware components such as GPIO or sensors, perform data processing such as image analysis, or integrate with MAVLink-based flight controllers. Once you integrate your module using the DataBus library, it appears in the DroneEngage system and is accessible from remote units and the DroneEngage Web Client.

## Architecture Overview

The DataBus framework follows a four-layer architecture:

```
┌─────────────────────────────────────┐
│  Your Application                   │
│  (C++ / Python / Node.js)           │
└──────────────────┬──────────────────┘
                   │ uses
                   ▼
┌─────────────────────────────────────┐
│  DataBus Library                    │
│  ┌─────────────┐  ┌──────────────┐  │
│  │  CModule    │  │ CFacade_Base │  │
│  │ (routing)   │  │  (send API)  │  │
│  └──────┬──────┘  └──────────────┘  │
│         │ CAndruavMessageParserBase │
│         │  (receive / dispatch)     │
└─────────┼───────────────────────────┘
          │
          ▼
┌─────────────────────────────────────┐
│  CUDPClient                         │
│  (chunking · reassembly · ID bcast) │
└──────────────────┬──────────────────┘
                   │ UDP
                   ▼
┌─────────────────────────────────────┐
│  de_comm Communicator (port 60000)  │
└─────────────────────────────────────┘
```

### Key Components

- **`CModule`** — Singleton manager for module registration, message routing, and the send API (`sendJMSG`, `sendBMSG`, `sendSYSMSG`, `sendMREMSG`).
- **`CFacade_Base`** — High-level send helpers (`sendErrorMessage`, `requestID`, `API_sendConfigTemplate`). Subclass to add domain-specific methods.
- **`CAndruavMessageParserBase`** — Abstract base class for inbound message dispatch. Subclass and override `parseCommand` / `parseRemoteExecute` to handle commands.
- **`CUDPClient`** — Low-level UDP transport with automatic chunking, chunk reassembly, and periodic module-ID broadcasting.

## Module Identity Fields

Every module is described by three string fields passed to `defineModule`:

<table>
<thead>
<tr><th>Field</th><th>Role</th><th>Example</th></tr>
</thead>
<tbody>
<tr><td><code>module_class</code></td><td>Type of module (from <code>MODULE_CLASS_*</code> constants)</td><td><code>"gen"</code>, <code>"fcb"</code>, <code>"camera"</code></td></tr>
<tr><td><code>module_id</code></td><td>Human-readable display name</td><td><code>"MyGPSSensor"</code></td></tr>
<tr><td><code>module_key</code></td><td>Unique GUID for this running instance (persisted across restarts)</td><td><code>"123456789012"</code></td></tr>
</tbody>
</table>

## Binary Message Wire Format

Binary messages have a composite structure:

```
[ JSON header (UTF-8) ] [ 0x00 ] [ binary payload ]
```

The JSON header carries routing metadata and the `ANDRUAV_PROTOCOL_MESSAGE_CMD` field. The null byte separates it from the raw binary payload (image, file, etc.).

## Initialization Sequence

All implementations follow the same order:

```
defineModule(class, id, key, version, filter)
  → addModuleFeatures(...)        [optional]
  → setHardware(serial, type)     [optional]
  → setMessageOnReceive(callback) [if receiving]
  → init(serverIP, serverPort, listenIP, listenPort, chunkSize)
```

After `init`, the UDP client starts two threads:
- **Receiver thread** — reassembles incoming chunks and delivers complete messages.
- **ID broadcaster thread** — sends the module identification packet once per second until the communicator acknowledges registration.

On first receipt of a `TYPE_AndruavModule_ID` response, `m_party_id` and `m_group_id` are populated with the values assigned by the communicator.

## Message Routing Types

<table>
<thead>
<tr><th>Constant</th><th>Direction</th></tr>
</thead>
<tbody>
<tr><td><code>CMD_TYPE_INTERMODULE</code></td><td>Internal — between local modules only</td></tr>
<tr><td><code>CMD_COMM_GROUP</code></td><td>Broadcast to the entire group</td></tr>
<tr><td><code>CMD_COMM_INDIVIDUAL</code></td><td>Direct to a specific <code>targetPartyID</code></td></tr>
<tr><td><code>CMD_COMM_SYSTEM</code></td><td>System-level communicator messages</td></tr>
</tbody>
</table>

## Core Features

All language implementations provide:

- **Module Registration** — Register your module with the DroneEngage communicator
- **JSON Messaging** — Send and receive structured JSON messages
- **Binary Data Support** — Transmit images, files, and other binary data
- **Message Chunking** — Automatic splitting (send) and reassembly (receive) of large messages
- **Message Filtering** — Subscribe to specific message types; empty filter = receive all
- **Thread-Safe Operations** — Mutex-protected concurrent message handling
- **Periodic ID Broadcasting** — Automatic module identification
- **High-Level Facade API** — Simplified interface for common operations
- **Extensible Parser** — Abstract `parseCommand` / `parseRemoteExecute` hooks

## Quick Start

Choose your preferred language:

- [C++ Implementation](./de-custom-plugins-cpp.md)
- [Node.js Implementation](./de-custom-plugins-nodejs.md)
- [Python Implementation](./de-custom-plugins-python.md)

## Use Cases

- **Hardware Integration** — GPIO, sensors, actuators
- **Data Processing** — Image processing, telemetry analysis
- **Custom Telemetry** — Send custom sensor data to the WebClient
- **MAVLink Integration** — Process and forward MAVLink messages (see `droneengage_mavlink` for a full example)
- **Remote Control** — Implement custom control logic
- **Monitoring** — Real-time system monitoring
- **Logging** — Custom data logging modules

## Technical Requirements

### Network Configuration

- Default `de_comm` port: `60000`
- Module listen ports: any unused UDP port (e.g., 61111–61234)
- Default UDP chunk size: `8192` bytes (`DEFAULT_UDP_DATABUS_PACKET_SIZE`)
- Maximum UDP chunk size: `65535` bytes (`MAX_UDP_DATABUS_PACKET_SIZE`)

### Language Standards

- **C++**: C++17, namespace `de::comm`, `CModule` / `CFacade_Base` singletons
- **Python**: Python 3.7+, snake_case methods, `__new__`-based singletons, `threading.RLock`
- **Node.js**: Node.js 12+, camelCase, constructor-based singleton, `async-lock` for thread safety
