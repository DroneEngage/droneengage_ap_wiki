# Python Plugin Development

The Python implementation provides a Pythonic interface to the DroneEngage DataBus using the same UDP-based protocol as the C++ and Node.js versions. It is ideal for rapid prototyping, scripting, sensor integration, and AI/ML pipelines. Source files are in `droneengage_databus/python/`.

## Features

- **Python 3.7+** — Tested with standard library only (`colorama` optional)
- **`__new__`-based Singleton** — `CModule()` always returns the same instance; thread-safe via `threading.Lock`
- **snake_case methods** — `add_module_features`, `set_hardware`, `send_error_message`, etc.
- **`m_OnReceive` callback** — Direct function reference, same pattern as C++
- **Binary Message Support** — `sendBMSG` with bytes payload
- **Config File Support** — `ConfigFile` and `LocalConfigFile` with hot-reload and C-style comment parsing
- **Extensible Parser** — Subclass `AndruavMessageParserBase` to handle commands
- **Colored Console Output** — `Colors` utility for styled terminal messages

---

## Quick Start

### Installation

```bash
cd droneengage_databus/python
pip install colorama       # optional — only needed for colored output
```

As an editable package:

```bash
pip install -e .
```

### Run

```bash
python python_client.py --help
python python_client.py MyModule 60000 61111    # name, de_comm port, listen port
python python_client.py MyModule               # uses defaults (60000 / 61111)
```

### Minimal Working Module

```python
from de_module import CModule
from de_facade_base import CFacade_Base
from messages import *

DEFAULT_UDP_DATABUS_PACKET_SIZE = 8192

c_module   = CModule()
base_facade = CFacade_Base()
base_facade.set_module(c_module)

c_module.defineModule(
    "gen",             # MODULE_CLASS_GENERIC
    "MyModule",        # module_id (display name)
    "unique-key-001",  # module_key (persistent GUID)
    "1.0.0",           # version
    []                 # message_filter (empty = receive nothing)
)

c_module.add_module_features("T")   # MODULE_FEATURE_SENDING_TELEMETRY
c_module.add_module_features("R")   # MODULE_FEATURE_RECEIVING_TELEMETRY

c_module.init("0.0.0.0", 60000, "0.0.0.0", 61111, DEFAULT_UDP_DATABUS_PACKET_SIZE)

import time
while True:
    time.sleep(1)
    base_facade.send_error_message("", ERROR_USER_DEFINED, NOTIFICATION_TYPE_NOTICE,
                                   NOTIFICATION_TYPE_INFO, "Hello from Python")
```

---

## Core Concepts

### Singleton Pattern

`CModule` uses `__new__` for thread-safe singleton access:

```python
c_module_a = CModule()
c_module_b = CModule()
assert c_module_a is c_module_b   # True — same instance
```

### Receive Callback

```python
def on_receive(message, msg_len, j_msg):
    msg_type = j_msg.get(ANDRUAV_PROTOCOL_MESSAGE_TYPE)
    cmd      = j_msg.get(ANDRUAV_PROTOCOL_MESSAGE_CMD, {})
    # dispatch on msg_type ...

c_module.m_OnReceive = on_receive
```

### Callback Chain

```
CUDPClient (chunk reassembly)
  → CModule.onReceive(message, len)
      → validates routing / handles TYPE_AndruavModule_ID (sets m_party_id / m_group_id)
      → calls m_OnReceive(message, len, jMsg)    ← your callback
          → optional: parser.parse_message(jMsg, message, len)
              → parse_command() / parse_remote_execute()  ← your overrides
```

### Thread Safety

All send operations use `threading.RLock`:

```python
with self.m_lock:
    # build message, call sendMSG
```

The singleton `__new__` gate uses a class-level `threading.Lock`:

```python
class CModule:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

---

## Complete API Reference

### `CModule`

```python
from de_module import CModule
c_module = CModule()   # singleton
```

#### Initialization

```python
def defineModule(self, module_class, module_id, module_key, module_version, message_filter):
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>module_class</code></td><td><code>"gen"</code>, <code>"fcb"</code>, <code>"camera"</code>, etc.</td></tr>
<tr><td><code>module_id</code></td><td>Display name shown in WebClient</td></tr>
<tr><td><code>module_key</code></td><td>Unique persistent GUID</td></tr>
<tr><td><code>module_version</code></td><td>Version string e.g. <code>"1.0.0"</code></td></tr>
<tr><td><code>message_filter</code></td><td>List of <code>TYPE_*</code> ints; <code>[]</code> = receive none</td></tr>
</tbody>
</table>

```python
def init(self, target_ip, broadcasts_port, host, listening_port, chunk_size):
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
<tr><td><code>chunk_size</code></td><td>Max UDP payload (use <code>8192</code>)</td></tr>
</tbody>
</table>

```python
def uninit(self):   # stops UDP threads; call before exit
```

#### Module Configuration

```python
def add_module_features(self, feature: str):     # "R", "T", "C", "V", "G", "A", "K", "P"
def set_hardware(self, hardware_serial: str, hardware_serial_type: int):  # 0=undef, 1=CPU
def appendExtraField(self, name: str, ms):        # add custom fields to ID broadcast
```

#### Sending

```python
def sendJMSG(self, targetPartyID: str, jmsg: dict, andruav_message_id: int, internal_message: bool):
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>targetPartyID</code></td><td><code>""</code> = broadcast; or specific party ID</td></tr>
<tr><td><code>jmsg</code></td><td>Dict — the message payload</td></tr>
<tr><td><code>andruav_message_id</code></td><td><code>TYPE_*</code> constant from <code>messages.py</code></td></tr>
<tr><td><code>internal_message</code></td><td><code>True</code> = intermodule; <code>False</code> = group/individual</td></tr>
</tbody>
</table>

```python
def sendBMSG(self, targetPartyID, bmsg, bmsg_length, andruav_message_id, internal_message, message_cmd):
```

<table>
<thead>
<tr><th>Parameter</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>bmsg</code></td><td><code>bytes</code> — binary payload</td></tr>
<tr><td><code>bmsg_length</code></td><td>Length of <code>bmsg</code> (or <code>0</code> for JSON-only)</td></tr>
<tr><td><code>message_cmd</code></td><td>Metadata dict placed in the JSON header</td></tr>
</tbody>
</table>

Wire format: `JSON_header.encode() + b'\0' + bmsg`

```python
def send_sys_msg(self, jmsg, andruav_message_id):   # system message to communicator
def sendMREMSG(self, command_type):                  # module remote-execute command
def forwardMSG(self, message, datalength):           # forward raw bytes
def sendMSG(self, msg, length):                      # low-level raw send
```

#### Receiving

```python
# Assign your callback:
c_module.m_OnReceive = lambda message, msg_len, j_msg: ...
```

#### State

```python
c_module.m_party_id   # set after communicator responds
c_module.m_group_id   # set after communicator responds
c_module.m_module_key
```

---

### `CFacade_Base` (alias `FacadeBase`)

```python
from de_facade_base import CFacade_Base
facade = CFacade_Base()   # singleton
facade.set_module(c_module)
```

#### Methods

```python
def set_module(self, module):   # inject CModule reference

def request_id(self, target_party_id: str):

def send_error_message(self, target_party_id: str, error_number: int,
                       info_type: int, notification_type: int, description: str):
```

<table>
<thead>
<tr><th><code>notification_type</code></th><th>Value</th></tr>
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

```python
def api_send_config_template(self, target_party_id: str, module_key: str,
                             json_file_content_json: dict, reply: bool):
```

#### Extending `CFacade_Base`

```python
from de_facade_base import CFacade_Base
from messages import TYPE_AndruavMessage_DUMMY

class MySensorFacade(CFacade_Base):
    def send_sensor_reading(self, target_party_id: str, value: float):
        import time
        self._module.sendJMSG(target_party_id,
            {"sensor_value": value, "ts": int(time.time() * 1e6)},
            TYPE_AndruavMessage_DUMMY, False)

facade = MySensorFacade()
facade.set_module(c_module)
facade.send_sensor_reading("", 23.5)
```

---

### `AndruavMessageParserBase` (alias `CAndruavMessageParserBase`)

Abstract base for inbound message dispatch. Subclass and override the two abstract methods.

```python
from de_message_parser_base import AndruavMessageParserBase

class MyParser(AndruavMessageParserBase):
    def __init__(self, facade):
        super().__init__()
        self._facade = facade

    def parse_remote_execute(self, andruav_message: dict):
        # handle TYPE_AndruavMessage_RemoteExecute
        pass

    def parse_command(self, andruav_message, full_message, full_message_length,
                      message_type, permission):
        from messages import TYPE_AndruavMessage_GPS
        if message_type == TYPE_AndruavMessage_GPS:
            cmd = andruav_message.get(ANDRUAV_PROTOCOL_MESSAGE_CMD, {})
            print(f"GPS: {cmd.get('lat')}, {cmd.get('lng')}")
```

**Wire up to receive:**

```python
parser = MyParser(facade)
c_module.m_OnReceive = lambda message, msg_len, j_msg: \
    parser.parse_message(j_msg, message, msg_len)
```

**State properties:**

```python
parser.is_binary        # True if message contains binary payload
parser.is_system        # True if sender is the communicator server
parser.is_inter_module  # True if routing type == CMD_TYPE_INTERMODULE
```

`parse_message` automatically handles `TYPE_AndruavMessage_CONFIG_ACTION` (restart, apply-config, fetch-config-template) before calling `parse_command`.

---

### Module Constants

```python
# Module classes (de_module.py)
MODULE_CLASS_GENERIC = "gen"
MODULE_CLASS_FCB     = "fcb"
MODULE_CLASS_VIDEO   = "camera"
MODULE_CLASS_P2P     = "p2p"
MODULE_CLASS_COMM    = "comm"

# Module features
MODULE_FEATURE_RECEIVING_TELEMETRY = "R"
MODULE_FEATURE_SENDING_TELEMETRY   = "T"
MODULE_FEATURE_CAPTURE_IMAGE       = "C"
MODULE_FEATURE_CAPTURE_VIDEO       = "V"

# Hardware types
HARDWARE_TYPE_UNDEFINED = 0
HARDWARE_TYPE_CPU       = 1
```

---

### `ConfigFile`

Reads a JSON configuration file with C-style comment support (`//` and `/* */`), file monitoring, backup creation, and hot-reload.

```python
from configFile import ConfigFile

config = ConfigFile.get_instance()
config.init_config_file("mymodule.config.json")
json_data = config.GetConfigJSON()
port = json_data.get("s2s_udp_target_port", "60000")
```

### `LocalConfigFile`

Stores per-instance data (e.g., the persistent `module_key`) in a local file.

```python
from localConfigFile import LocalConfigFile

local_config = LocalConfigFile.get_instance()
local_config.InitConfigFile("mymodule.local")
module_key = local_config.getStringField("module_key")
if not module_key:
    import time
    module_key = str(int(time.time() * 1e6))
    local_config.addStringField("module_key", module_key)
    local_config.apply()
```

---

## Package Structure

```
droneengage_databus/python/
├── __init__.py               # Package init with re-exports
├── de_module.py              # CModule — main module interface
├── udpClient.py              # CUDPClient — UDP with chunking/reassembly
├── de_facade_base.py         # FacadeBase (CFacade_Base) — high-level send API
├── de_message_parser_base.py # AndruavMessageParserBase — abstract parser
├── configFile.py             # ConfigFile — JSON config with hot-reload
├── localConfigFile.py        # LocalConfigFile — persistent local settings
├── messages.py               # All TYPE_* and protocol constants
├── colors.py / console_colors.py  # ANSI color helpers
└── python_client.py          # Working example client
```

---

## Message Handling

### Message Filter

```python
from messages import TYPE_AndruavMessage_GPS, TYPE_AndruavMessage_MAVLINK

c_module.defineModule("gen", "GPSMonitor", module_key, "1.0.0",
    [TYPE_AndruavMessage_GPS, TYPE_AndruavMessage_MAVLINK])
```

Empty `[]` = receive no messages. `None` or omit = receive all.

### Dispatch Pattern

```python
from messages import (ANDRUAV_PROTOCOL_MESSAGE_TYPE, ANDRUAV_PROTOCOL_MESSAGE_CMD,
                      TYPE_AndruavMessage_GPS)

def on_receive(message, msg_len, j_msg):
    try:
        msg_type = j_msg.get(ANDRUAV_PROTOCOL_MESSAGE_TYPE)
        cmd      = j_msg.get(ANDRUAV_PROTOCOL_MESSAGE_CMD, {})
        if msg_type == TYPE_AndruavMessage_GPS:
            print(f"GPS: lat={cmd.get('lat')}, lng={cmd.get('lng')}")
    except Exception as e:
        print(f"Error: {e}")

c_module.m_OnReceive = on_receive
```

---

## Binary Transmission

```python
from messages import TYPE_AndruavMessage_IMG
import time

with open("photo.jpg", "rb") as f:
    image_data = f.read()

metadata = {"lat": 31.5, "lng": 34.5, "alt": 100,
            "tim": int(time.time() * 1e6)}

c_module.sendBMSG("", image_data, len(image_data),
                  TYPE_AndruavMessage_IMG, False, metadata)
```

---

## Custom Message Types

```python
from messages import TYPE_AndruavMessage_USER_RANGE_START

TYPE_MY_SENSOR_DATA = TYPE_AndruavMessage_USER_RANGE_START + 0
TYPE_MY_CMD_ACK     = TYPE_AndruavMessage_USER_RANGE_START + 1

c_module.sendJMSG("", {"temperature": 25.5}, TYPE_MY_SENSOR_DATA, False)
```

---

## Configuration Management

### Reading Config Values

```python
from configFile import ConfigFile

config   = ConfigFile.get_instance()
config.init_config_file("mymodule.config.json")
json_cfg = config.GetConfigJSON()

target_ip   = json_cfg.get("s2s_udp_target_ip", "0.0.0.0")
target_port = int(json_cfg.get("s2s_udp_target_port", 60000))
listen_port = int(json_cfg.get("s2s_udp_listening_port", 61111))

c_module.init(target_ip, target_port, "0.0.0.0", listen_port, 8192)
```

### Hot-Reload Monitoring

`ConfigFile` supports watching the config file for changes:

```python
config.startMonitoring()
```

### Config Template

The parser's `parse_command` can serve a `template.json` to the WebClient:

```python
# Handled automatically by AndruavMessageParserBase._handle_config_action
# when TYPE_AndruavMessage_CONFIG_ACTION with action CONFIG_REQUEST_FETCH_CONFIG_TEMPLATE
# is received — it reads template.json and calls:
self._facade.api_send_config_template(sender, module_key, json_content, True)
```

---

## Graceful Shutdown

```python
import signal, sys

def signal_handler(signum, frame):
    c_module.uninit()
    sys.exit(0)

signal.signal(signal.SIGINT,  signal_handler)
signal.signal(signal.SIGTERM, signal_handler)
```

---

## Console Colors

```python
from console_colors import Colors

print(Colors.INFO_CONSOLE_BOLD_TEXT + "Starting module..." + Colors.NORMAL_CONSOLE_TEXT)
print(Colors.SUCCESS_CONSOLE_BOLD_TEXT + "Connected!" + Colors.NORMAL_CONSOLE_TEXT)
print(Colors.ERROR_CONSOLE_TEXT + "Connection failed" + Colors.NORMAL_CONSOLE_TEXT)
```

---

## Best Practices

1. **Persistent module key** — Use `LocalConfigFile` to generate and store a `module_key` on first run.
2. **Unique listen port** — Each module instance must bind a different port.
3. **Minimal message filter** — List only `TYPE_*` values you handle.
4. **Wrap `m_OnReceive` in try/except** — Exceptions inside will silently stop message delivery.
5. **Call `uninit()` on shutdown** — Stops UDP threads and sockets cleanly.
6. **Use `CFacade_Base`** — Extend it rather than calling `sendJMSG` directly from application code.
7. **`add_module_features`** uses snake_case — not `addModuleFeatures`.

---

## Troubleshooting

<table>
<thead>
<tr><th>Issue</th><th>Solution</th></tr>
</thead>
<tbody>
<tr><td>Module not in WebClient Details</td><td>Check <code>c_module.m_party_id</code> is non-empty after init</td></tr>
<tr><td>Messages not received</td><td>Verify <code>TYPE_*</code> is in <code>message_filter</code> list</td></tr>
<tr><td><code>ModuleNotFoundError: messages</code></td><td>Run from <code>python/</code> directory or install with <code>pip install -e .</code></td></tr>
<tr><td>Binary data corrupt</td><td>Verify <code>bmsg_length == len(bmsg)</code></td></tr>
<tr><td><code>m_OnReceive</code> not called</td><td>Ensure <code>c_module.init(...)</code> was called after <code>m_OnReceive</code> was set</td></tr>
<tr><td>Config not found</td><td>Pass absolute or cwd-relative path to <code>init_config_file</code></td></tr>
</tbody>
</table>

---

## See Also

- [Main custom plugins page](./de-custom-plugins.md)
- [C++ implementation](./de-custom-plugins-cpp.md)
- [Node.js implementation](./de-custom-plugins-nodejs.md)
- [Python source](https://github.com/DroneEngage/droneengage_databus/tree/master/python)
- [Python threading docs](https://docs.python.org/3/library/threading.html)
