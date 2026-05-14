# Node.js Plugin Development

The Node.js implementation provides an asynchronous interface to the DroneEngage DataBus using the same UDP-based protocol as the C++ and Python versions. It is ideal for web-integrated solutions, rapid prototyping, and Node.js-based automation. Source files are in `droneengage_databus/nodejs/`.

## Features

- **Constructor Singleton** — `new CModule()` always returns the same instance
- **`async-lock`** — Reentrant lock for thread-safe `sendJMSG` / `sendBMSG`
- **`m_OnReceive` callback** — Direct function reference; no EventEmitter overhead
- **Binary Data Support** — `sendBMSG` with `Buffer` payload
- **Message Chunking** — Automatic split and reassembly via `CUDPClient`
- **Extensible Parser** — Subclass `AndruavMessageParserBase` to handle commands
- **Graceful Shutdown** — SIGINT/SIGTERM handlers with `uninit()`

---

## Quick Start

### Installation

```bash
cd droneengage_databus/nodejs
npm install
```

Required packages (`package.json`):
- `async-lock` ^1.4.1 — reentrant locking for send operations
- `readline` ^1.3.0 — interactive console input

### Run

```bash
node client.js --help
node client.js MyModule 60000 61111    # name, de_comm port, listen port
node client.js MyModule               # uses defaults (60000 / 61111)
```

### Minimal Working Module

```javascript
const CModule = require('./de_module');
const { CFacade_Base } = require('./de_facade_base');
const { TYPE_AndruavMessage_DUMMY } = require('./messages');

const DEFAULT_UDP_DATABUS_PACKET_SIZE = 8192;

const cModule = new CModule();
const facade = new CFacade_Base();
facade.setModule(cModule);

cModule.defineModule(
    "gen",             // MODULE_CLASS_GENERIC
    "MyModule",        // module_id (display name)
    "unique-key-001",  // module_key (persistent GUID)
    "1.0.0",           // version
    []                 // message_filter (empty = receive nothing)
);

cModule.addModuleFeatures("T");  // MODULE_FEATURE_SENDING_TELEMETRY
cModule.addModuleFeatures("R");  // MODULE_FEATURE_RECEIVING_TELEMETRY

cModule.init("0.0.0.0", 60000, "0.0.0.0", 61111, DEFAULT_UDP_DATABUS_PACKET_SIZE);

setInterval(() => {
    cModule.sendJMSG("", { t: "hello" }, TYPE_AndruavMessage_DUMMY, true);
}, 1000);
```

---

## Core Concepts

### Singleton Pattern

`CModule` uses a constructor-based singleton:

```javascript
const cModule = new CModule();  // always returns the same instance
```

### Receive Callback

Messages are delivered via a property assignment — **not** via `EventEmitter`:

```javascript
cModule.m_OnReceive = (message, length, jMsg) => {
    const msgType = jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE];
    const cmd     = jMsg[ANDRUAV_PROTOCOL_MESSAGE_CMD];
    // dispatch on msgType ...
};
```

### Callback Chain

```
CUDPClient (chunk reassembly)
  → CModule.onReceive(message, len)
      → validates routing / handles TYPE_AndruavModule_ID (sets m_party_id / m_group_id)
      → calls m_OnReceive(message, len, jMsg)   ← your callback
          → optional: parser.parseMessage(jMsg, message, len)
              → parseCommand() / parseRemoteExecute()  ← your overrides
```

### `async-lock` Thread Safety

`sendJMSG` and `sendBMSG` acquire a named lock before building and sending:

```javascript
this.m_lock.acquire('lock', (done) => {
    // build message, call sendMSG
    done();
});
```

---

## Complete API Reference

### `CModule` class

```javascript
const CModule = require('./de_module');
const cModule = new CModule();
```

#### Initialization

```javascript
defineModule(module_class, module_id, module_key, module_version, message_filter)
```

<table>
<thead>
<tr><th>Parameter</th><th>Type</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>module_class</code></td><td>string</td><td><code>"gen"</code>, <code>"fcb"</code>, <code>"camera"</code>, etc.</td></tr>
<tr><td><code>module_id</code></td><td>string</td><td>Display name shown in WebClient</td></tr>
<tr><td><code>module_key</code></td><td>string</td><td>Unique persistent GUID</td></tr>
<tr><td><code>module_version</code></td><td>string</td><td>Version string e.g. <code>"1.0.0"</code></td></tr>
<tr><td><code>message_filter</code></td><td>number[]</td><td>Array of <code>TYPE_*</code> ints; <code>[]</code> = receive none</td></tr>
</tbody>
</table>

```javascript
init(target_ip, broadcasts_port, host, listening_port, chunk_size)
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>target_ip</code></td><td>Communicator IP (<code>"0.0.0.0"</code> = localhost)</td></tr>
<tr><td><code>broadcasts_port</code></td><td>Communicator port (typically <code>60000</code>)</td></tr>
<tr><td><code>host</code></td><td>Local bind address (<code>"0.0.0.0"</code>)</td></tr>
<tr><td><code>listening_port</code></td><td>Local receive port (unique per module)</td></tr>
<tr><td><code>chunk_size</code></td><td>Max UDP payload (<code>DEFAULT_UDP_DATABUS_PACKET_SIZE = 8192</code>)</td></tr>
</tbody>
</table>

```javascript
uninit()   // stops UDP client; call before process.exit()
```

#### Module Configuration

```javascript
addModuleFeatures(feature)      // "R", "T", "C", "V", "G", "A", "K", "P"
setHardware(hardware_serial, hardware_serial_type)  // serial string, 0=undef / 1=CPU
appendExtraField(name, value)   // add custom fields to the ID broadcast packet
```

#### Sending

```javascript
// Send JSON message
sendJMSG(targetPartyID, jmsg, andruav_message_id, internal_message)
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>targetPartyID</code></td><td><code>""</code> = broadcast; or specific party ID</td></tr>
<tr><td><code>jmsg</code></td><td>Plain object — the message payload</td></tr>
<tr><td><code>andruav_message_id</code></td><td><code>TYPE_*</code> constant from <code>messages.js</code></td></tr>
<tr><td><code>internal_message</code></td><td><code>true</code> = intermodule; <code>false</code> = group/individual</td></tr>
</tbody>
</table>

```javascript
// Send binary message (JSON header + null byte + binary payload)
sendBMSG(targetPartyID, bmsg, bmsg_length, andruav_message_id, internal_message, message_cmd)
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>bmsg</code></td><td><code>Buffer</code> containing binary data</td></tr>
<tr><td><code>bmsg_length</code></td><td>Length of <code>bmsg</code></td></tr>
<tr><td><code>message_cmd</code></td><td>Metadata object placed in the JSON header</td></tr>
</tbody>
</table>

```javascript
sendSysMsg(jmsg, andruav_message_id)     // system message to communicator
sendMREMSG(command_type)                 // module remote-execute command
forwardMSG(message, datalength)          // forward a raw buffer
sendMSG(msg, length)                     // low-level raw send
```

#### Receiving

```javascript
// Assign your callback:
cModule.m_OnReceive = (message, length, jMsg) => { ... };
```

**`jMsg` fields:**

<table>
<thead>
<tr><th>Constant</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>ANDRUAV_PROTOCOL_MESSAGE_TYPE</code></td><td>Message type integer</td></tr>
<tr><td><code>ANDRUAV_PROTOCOL_MESSAGE_CMD</code></td><td>Payload object</td></tr>
<tr><td><code>INTERMODULE_ROUTING_TYPE</code></td><td>Routing type string</td></tr>
<tr><td><code>ANDRUAV_PROTOCOL_SENDER</code></td><td>Sender's party ID</td></tr>
</tbody>
</table>

#### State

```javascript
cModule.m_party_id   // set after communicator responds to registration
cModule.m_group_id   // set after communicator responds to registration
cModule.m_module_key
```

---

### `CFacade_Base` class

```javascript
const { CFacade_Base } = require('./de_facade_base');
const facade = new CFacade_Base();
facade.setModule(cModule);
```

#### Methods

```javascript
setModule(module)   // inject CModule reference

requestID(targetPartyID)   // request ID from target

sendErrorMessage(targetPartyID, errorNumber, infoType, notificationType, description)
```

<table>
<thead>
<tr><th><code>notificationType</code></th><th>Value</th></tr>
</thead>
<tbody>
<tr><td><code>NOTIFICATION_TYPE_EMERGENCY</code></td><td>0</td></tr>
<tr><td><code>NOTIFICATION_TYPE_ALERT</code></td><td>1</td></tr>
<tr><td><code>NOTIFICATION_TYPE_CRITICAL</code></td><td>2</td></tr>
<tr><td><code>NOTIFICATION_TYPE_ERROR</code></td><td>3</td></tr>
<tr><td><code>NOTIFICATION_TYPE_WARNING</code></td><td>4</td></tr>
<tr><td><code>NOTIFICATION_TYPE_NOTICE</code></td><td>5</td></tr>
<tr><td><code>NOTIFICATION_TYPE_INFO</code></td><td>6</td></tr>
<tr><td><code>NOTIFICATION_TYPE_DEBUG</code></td><td>7</td></tr>
</tbody>
</table>

```javascript
API_sendConfigTemplate(target_party_id, module_key, json_file_content_json, reply)
```

#### Extending `CFacade_Base`

```javascript
// See my_facade.js for the pattern:
const { CFacade_Base } = require('./de_facade_base');

class CMyFacade extends CFacade_Base {
    constructor(module) {
        super();
        if (module) this.setModule(module);
    }

    sendSensorReading(targetPartyID, value) {
        this.m_module.sendJMSG(targetPartyID,
            { sensor_value: value, ts: Date.now() },
            TYPE_AndruavMessage_DUMMY, false);
    }
}

module.exports = { CMyFacade };
```

Usage in `client.js`:

```javascript
const cModule  = new CModule();
const myFacade = new CMyFacade(cModule);
myFacade.sendErrorMessage("", ERROR_USER_DEFINED, NOTIFICATION_TYPE_INFO,
                          NOTIFICATION_TYPE_INFO, "Hello from Node.js");
```

---

### `AndruavMessageParserBase` class

Abstract base for inbound message dispatch. Subclass and override the two abstract methods.

```javascript
const { AndruavMessageParserBase } = require('./de_message_parser_base');
```

```javascript
// Call from your m_OnReceive callback:
parseMessage(andruav_message, full_message, full_message_length)
```

**Must override:**

```javascript
parseRemoteExecute(andruav_message)   // called when TYPE_AndruavMessage_RemoteExecute
parseCommand(andruav_message, full_message, full_message_length, messageType, permission)
```

**State getters:**

```javascript
get isBinary()      // true if message contains binary payload
get isSystem()      // true if sender is the communicator server
get isInterModule() // true if routing type == CMD_TYPE_INTERMODULE
```

`parseMessage` automatically handles `TYPE_AndruavMessage_CONFIG_ACTION` before calling `parseCommand`.

**Example subclass:**

```javascript
const { AndruavMessageParserBase } = require('./de_message_parser_base');
const { TYPE_AndruavMessage_MAVLINK } = require('./messages');

class MyParser extends AndruavMessageParserBase {
    constructor(facade) {
        super();
        this._facade = facade;
    }

    parseRemoteExecute(andruav_message) {
        // handle remote execute commands
    }

    parseCommand(andruav_message, full_message, full_message_length, messageType, permission) {
        if (messageType === TYPE_AndruavMessage_MAVLINK) {
            const mavlinkData = Buffer.from(full_message).slice(/* header offset */);
            // process mavlink bytes
        }
    }
}
```

**Wire up to receive:**

```javascript
const parser = new MyParser(myFacade);
cModule.m_OnReceive = (message, length, jMsg) => {
    parser.parseMessage(jMsg, message, length);
};
```

---

### Module Class Constants

From `messages.js`:

```javascript
const MODULE_CLASS_GENERIC = "gen";
const MODULE_CLASS_FCB     = "fcb";
const MODULE_CLASS_VIDEO   = "camera";
const MODULE_CLASS_P2P     = "p2p";
const MODULE_CLASS_COMM    = "comm";
```

### Module Feature Constants

```javascript
const MODULE_FEATURE_RECEIVING_TELEMETRY = "R";
const MODULE_FEATURE_SENDING_TELEMETRY   = "T";
const MODULE_FEATURE_CAPTURE_IMAGE       = "C";
const MODULE_FEATURE_CAPTURE_VIDEO       = "V";
const MODULE_FEATURE_GPIO                = "G";
const MODULE_FEATURE_AI_RECOGNITION      = "A";
const MODULE_FEATURE_TRACKING            = "K";
const MODULE_FEATURE_P2P                 = "P";
```

---

## Message Handling

### Message Filter

Pass an array of `TYPE_*` integers to `defineModule`. The communicator only forwards matching message types to your module:

```javascript
const { TYPE_AndruavMessage_GPS, TYPE_AndruavMessage_MAVLINK } = require('./messages');

cModule.defineModule("gen", "GPSMonitor", moduleId, "1.0.0",
    [TYPE_AndruavMessage_GPS, TYPE_AndruavMessage_MAVLINK]);
```

An empty array `[]` means the module receives **no** messages (send-only). Pass `null` or omit to receive **all** messages.

### Receive Dispatch Pattern

```javascript
const { ANDRUAV_PROTOCOL_MESSAGE_TYPE, ANDRUAV_PROTOCOL_MESSAGE_CMD,
        TYPE_AndruavMessage_GPS, TYPE_AndruavMessage_POWER } = require('./messages');

cModule.m_OnReceive = (message, length, jMsg) => {
    try {
        const msgType = jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE];
        const cmd     = jMsg[ANDRUAV_PROTOCOL_MESSAGE_CMD];

        switch (msgType) {
            case TYPE_AndruavMessage_GPS:
                console.log(`GPS: ${cmd.lat}, ${cmd.lng}`);
                break;
            case TYPE_AndruavMessage_POWER:
                console.log(`Battery: ${cmd.voltage}V`);
                break;
        }
    } catch (e) {
        console.error('Error processing message:', e);
    }
};
```

---

## Binary Transmission

```javascript
const fs = require('fs');
const { TYPE_AndruavMessage_IMG } = require('./messages');

const imageData = fs.readFileSync('photo.jpg');
const metadata  = { lat: 31.5, lng: 34.5, alt: 100, tim: Date.now() * 1000 };

cModule.sendBMSG("", imageData, imageData.length,
                 TYPE_AndruavMessage_IMG, false, metadata);
```

---

## Custom Message Types

```javascript
const { TYPE_AndruavMessage_USER_RANGE_START } = require('./messages');

const TYPE_MY_SENSOR_DATA = TYPE_AndruavMessage_USER_RANGE_START + 0;
const TYPE_MY_CMD_ACK     = TYPE_AndruavMessage_USER_RANGE_START + 1;

cModule.sendJMSG("", { temperature: 25.5 }, TYPE_MY_SENSOR_DATA, false);
```

---

## Graceful Shutdown

```javascript
process.on('SIGINT',  () => cleanupAndExit());
process.on('SIGTERM', () => cleanupAndExit());

function cleanupAndExit() {
    try { cModule.uninit(); } catch (e) {}
    process.exit(0);
}
```

---

## TypeScript Type Definitions

```typescript
// types.d.ts
declare module './de_module' {
    export default class CModule {
        m_OnReceive: ((message: Buffer, length: number, jMsg: any) => void) | null;
        m_party_id: string;
        m_group_id: string;

        defineModule(module_class: string, module_id: string, module_key: string,
                     module_version: string, message_filter: number[]): void;
        init(target_ip: string, broadcasts_port: number, host: string,
             listening_port: number, chunk_size: number): boolean;
        uninit(): boolean;
        addModuleFeatures(feature: string): void;
        setHardware(hardware_serial: string, hardware_serial_type: number): void;
        sendJMSG(targetPartyID: string, jmsg: object,
                 andruav_message_id: number, internal_message: boolean): void;
        sendBMSG(targetPartyID: string, bmsg: Buffer, bmsg_length: number,
                 andruav_message_id: number, internal_message: boolean,
                 message_cmd: object): void;
        appendExtraField(name: string, value: any): void;
    }
}
```

---

## Port Configuration

- **Communicator port:** `60000` (default)
- **Module listen port:** any unused UDP port; `client.js` defaults to `61111`
- Use the range 61000–62000 for custom modules to avoid conflicts

---

## Debugging

Enable verbose output by setting `DEBUG` env var or uncommenting console.log lines in `de_module.js`:

```bash
DEBUG=* node client.js MyModule
```

Key log lines already present in the source:
```javascript
console.log(`RX MSG: len ${len}: ${message}`);      // de_module.js onReceive
console.log(`sendJMSG: ${msg}`);                     // de_module.js sendJMSG
```

---

## Best Practices

1. **Persistent module key** — Use a hardcoded or file-stored GUID so the communicator tracks the module across restarts.
2. **Unique listen port** — Each running instance needs its own port.
3. **Minimal message filter** — Subscribe only to types you handle.
4. **Wrap `m_OnReceive` in try-catch** — Exceptions inside the callback will silently stop message delivery.
5. **Call `uninit()` before exit** — Stops UDP threads cleanly.
6. **Use `CMyFacade`** — Extend `CFacade_Base` rather than calling `cModule.sendJMSG` directly from application code.

---

## Troubleshooting

<table>
<thead>
<tr><th>Issue</th><th>Solution</th></tr>
</thead>
<tbody>
<tr><td>Module not in WebClient Details</td><td>Verify communicator is on port 60000; check <code>cModule.m_party_id</code> is set</td></tr>
<tr><td>Messages not received</td><td>Verify <code>TYPE_*</code> is in <code>message_filter</code> array</td></tr>
<tr><td><code>Cannot find module 'async-lock'</code></td><td>Run <code>npm install</code> in the <code>nodejs/</code> directory</td></tr>
<tr><td>Module not reconnecting after communicator restart</td><td><code>m_FirstReceived</code> flag prevents repeat ID sends; restart the module</td></tr>
<tr><td>Binary data truncated</td><td>Ensure <code>bmsg_length === bmsg.length</code></td></tr>
</tbody>
</table>

---

## See Also

- [Main custom plugins page](./de-custom-plugins.md)
- [C++ implementation](./de-custom-plugins-cpp.md)
- [Python implementation](./de-custom-plugins-python.md)
- [Node.js source](https://github.com/DroneEngage/droneengage_databus/tree/master/nodejs)
- [Node.js EventEmitter docs](https://nodejs.org/api/events.html)

