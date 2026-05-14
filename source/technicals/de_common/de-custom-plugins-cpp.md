# C++ Plugin Development

The C++ implementation provides high-performance, low-latency communication with the DroneEngage DataBus. It is ideal for performance-critical applications, hardware integration, and production drone modules. The library lives in `droneengage_common/de_databus/` and is used by all C++ DroneEngage components including the full MAVLink bridge (`droneengage_mavlink`).

## Features

- **C++17 Standard** — Modern C++ with RAII, lambdas, and STL concurrency
- **Namespace `de::comm`** — All library classes under `de::comm::`
- **Singleton Pattern** — `CModule`, `CFacade_Base` are global singletons
- **Thread-Safe Send** — `std::mutex`-protected `sendJMSG` / `sendBMSG`
- **Automatic Chunking** — Large messages split and reassembled transparently
- **Config-File Integration** — `CConfigFile` / `CLocalConfigFile` for production deployments
- **Extensible Parser** — Subclass `CAndruavMessageParserBase` to handle any command

---

## Quick Start

### Build

```bash
cd droneengage_databus/client
mkdir build && cd build
cmake ..
make
```

### Run

```bash
./client --help                          # Show help
./client MyModule 60000 61111            # Module name, DE comm port, listen port
./client MyModule                        # Uses defaults (60000 / 61111)
```

### Minimal Working Module

```cpp
#include "de_common/de_databus/de_module.hpp"
using namespace de::comm;
using Json_de = nlohmann::json;

int main() {
    CModule& cModule = CModule::getInstance();

    cModule.defineModule(
        MODULE_CLASS_GENERIC,   // module_class
        "MyModule",             // module_id  (display name)
        "unique-key-001",       // module_key (persistent GUID)
        "1.0.0",                // version
        Json_de::array()        // message_filter (empty = receive nothing)
    );

    cModule.addModuleFeatures(MODULE_FEATURE_SENDING_TELEMETRY);
    cModule.addModuleFeatures(MODULE_FEATURE_RECEIVING_TELEMETRY);

    cModule.init("0.0.0.0", 60000, "0.0.0.0", 61111,
                 DEFAULT_UDP_DATABUS_PACKET_SIZE);

    while (true) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        Json_de msg = {{"t", "hello"}};
        cModule.sendJMSG("", msg, TYPE_AndruavMessage_DUMMY, true);
    }
}
```

---

## Core Concepts

### Namespace

All classes are under `de::comm::`. Either use:

```cpp
using namespace de;
using namespace de::comm;
```

or qualify fully: `de::comm::CModule::getInstance()`.

### Singleton Access

```cpp
de::comm::CModule&      cModule = de::comm::CModule::getInstance();
de::comm::CFacade_Base& cFacade = de::comm::CFacade_Base::getInstance();
```

### Callback Chain

```
CModule::onReceive()          ← called by CUDPClient after chunk reassembly
  → validates routing fields
  → handles TYPE_AndruavModule_ID (stores m_party_id / m_group_id)
  → calls m_OnReceive(message, len, jMsg)    ← your callback
      → your callback calls parser.parseMessage(jMsg, message, len)
          → parseRemoteExecute() or parseCommand()  ← your overrides
```

### Registration Flow

After `init()`, the ID broadcaster sends `TYPE_AndruavModule_ID` every second. On the first communicator response, `CModule::onReceive` stores:

```cpp
m_party_id  // accessible via cModule.getPartyId()
m_group_id  // accessible via cModule.getGroupId()
```

---

## Complete API Reference

### `de::comm::CModule`

Singleton module manager. Handles module registration, message routing, and send operations.

```cpp
static CModule& getInstance();
```

#### Initialization

```cpp
void defineModule(
    std::string module_class,    // MODULE_CLASS_* constant
    std::string module_id,       // Display name shown in WebClient
    std::string module_key,      // Unique persistent GUID
    std::string module_version,  // Version string, e.g. "1.0.0"
    Json_de     message_filter   // Array of TYPE_* ints; empty = receive none
);

bool init(
    const std::string targetIP,    // Communicator IP ("0.0.0.0" = localhost)
    int broadcatsPort,             // Communicator port (typically 60000)
    const std::string host,        // Local bind IP ("0.0.0.0")
    int listenningPort,            // Local listen port (unique per module)
    int chunkSize                  // Max UDP payload (DEFAULT_UDP_DATABUS_PACKET_SIZE)
);

bool uninit();   // Stop UDP threads and clean up
```

#### Module Configuration

```cpp
void addModuleFeatures(const std::string feature);   // MODULE_FEATURE_* constant
void setHardware(const std::string hardware_serial,
                 const ENUM_HARDWARE_TYPE hardware_serial_type);
void setModuleId(const std::string module_id);
void setModuleClass(const std::string module_class);
void setModuleKey(const char* module_key);
void appendExtraField(const std::string name, const Json_de& ms);
```

#### Sending

```cpp
void sendJMSG(
    const std::string targetPartyID,  // "" = broadcast; or specific party ID
    const Json_de jmsg,               // Message payload object
    const int andruav_message_id,     // TYPE_AndruavMessage_* constant
    const bool internal_message       // true = intermodule; false = group/individual
);

void sendBMSG(
    const std::string& targetPartyID,
    const char* bmsg,                 // Binary payload bytes
    const int bmsg_length,
    const int& andruav_message_id,
    const bool& internal_message,
    const Json_de& message_cmd        // Metadata JSON in the header
);

void sendSYSMSG(const Json_de& jmsg, const int& andruav_message_id);
void sendMREMSG(const int& command_type);
void forwardMSG(const char* message, const std::size_t datalength);
void sendMSG(const char* msg, const int length);  // low-level raw send
```

#### Receiving

```cpp
void setMessageOnReceive(void (*onReceive)(const char*, int len, Json_de jMsg));
```

Callback signature:
```cpp
void onReceive(const char* message, int len, Json_de jMsg) {
    const int msgType = jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE].get<int>();
    const Json_de& cmd = jMsg[ANDRUAV_PROTOCOL_MESSAGE_CMD];
    // dispatch on msgType ...
}
```

#### Getters

```cpp
const std::string getModuleKey() const;
const std::string getPartyId()  const;   // Set after communicator responds
const std::string getGroupId()  const;   // Set after communicator responds
const Json_de     getModuleFeatures() const;
```

---

### `de::comm::CUDPClient`

Low-level UDP transport — normally used indirectly through `CModule`.

```cpp
void init(const char* targetIP, int broadcatsPort,
          const char* host, int listenningPort, int chunkSize);
void start();
void stop();
void setJsonId(std::string jsonID);   // Sets the ID packet broadcast periodically
void sendMSG(const char* msg, const int length);
bool isStarted() const;
```

<table>
<thead>
<tr><th>Constant</th><th>Value</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td><code>DEFAULT_UDP_DATABUS_PACKET_SIZE</code></td><td><code>8192</code></td><td>Recommended chunk size</td></tr>
<tr><td><code>MAX_UDP_DATABUS_PACKET_SIZE</code></td><td><code>0xFFFF</code></td><td>Maximum allowed chunk size</td></tr>
</tbody>
</table>

**Chunking protocol:**
- Each chunk carries a 2-byte little-endian header (chunk sequence number).
- The final chunk is marked with header `0xFFFF`.
- ~10 ms delay between chunks to reduce packet loss under congestion.

---

### `de::comm::CFacade_Base`

Singleton. Holds `m_module` reference to `CModule`. Provides high-level send helpers. Subclass to add domain-specific methods.

```cpp
static CFacade_Base& getInstance();

void requestID(const std::string& target_party_id) const;

void sendErrorMessage(
    const std::string& target_party_id,
    const int& error_number,        // e.g. ERROR_USER_DEFINED
    const int& info_type,           // component reporting the error
    const int& notification_type,   // e.g. NOTIFICATION_TYPE_INFO
    const std::string& description
) const;

void API_sendConfigTemplate(
    const std::string& target_party_id,
    const std::string& module_key,
    const Json_de& json_file_content_json,
    const bool reply
);
```

**Extending `CFacade_Base`:**

```cpp
class CMyFacade : public de::comm::CFacade_Base {
public:
    static CMyFacade& getInstance() {
        static CMyFacade instance;
        return instance;
    }

    void sendSensorReading(const std::string& target, double value) const {
        Json_de msg = {{"sensor_value", value}, {"ts", get_time_usec()}};
        m_module.sendJMSG(target, msg, TYPE_AndruavMessage_DUMMY, false);
    }
};
```

---

### `de::comm::CAndruavMessageParserBase`

Abstract base for inbound message dispatch. Subclass and override the two pure-virtual methods.

```cpp
// Call from your m_OnReceive callback:
void parseMessage(Json_de& andruav_message, const char* full_message,
                  const int& full_message_length);
```

**Must override (pure virtual):**

```cpp
protected:
    virtual void parseRemoteExecute(Json_de& andruav_message) = 0;

    virtual void parseCommand(Json_de& andruav_message,
                              const char* full_message,
                              const int& full_message_length,
                              int messageType,
                              uint32_t permission) = 0;
```

**State flags set before dispatch:**

```cpp
protected:
    bool m_is_binary;        // true if message contains binary payload
    bool m_is_system;        // true if sender is the communicator server
    bool m_is_inter_module;  // true if routing type == CMD_TYPE_INTERMODULE

    de::comm::CFacade_Base& m_facade = de::comm::CFacade_Base::getInstance();
```

`parseMessage` automatically handles `TYPE_AndruavMessage_CONFIG_ACTION` (restart, apply-config, fetch-config-template) before calling `parseCommand`.

---

### Module Class Constants

```cpp
#define MODULE_CLASS_COMM          "comm"    // Communication module
#define MODULE_CLASS_FCB           "fcb"     // Flight control board
#define MODULE_CLASS_VIDEO         "camera"  // Camera / video
#define MODULE_CLASS_P2P           "p2p"     // Peer-to-peer
#define MODULE_CLASS_GENERIC       "gen"     // General purpose
#define MODULE_CLASS_GPIO          "gpio"    // GPIO hardware
#define MODULE_CLASS_A_RECOGNITION "ai_rec"  // AI recognition
#define MODULE_CLASS_TRACKING      "trk"     // Target tracking
#define MODULE_CLASS_VIEWLINK      "vlk"     // Viewlink camera
```

### Module Feature Constants

```cpp
#define MODULE_FEATURE_RECEIVING_TELEMETRY  "R"
#define MODULE_FEATURE_SENDING_TELEMETRY    "T"
#define MODULE_FEATURE_CAPTURE_IMAGE        "C"
#define MODULE_FEATURE_CAPTURE_VIDEO        "V"
#define MODULE_FEATURE_GPIO                 "G"
#define MODULE_FEATURE_AI_RECOGNITION       "A"
#define MODULE_FEATURE_TRACKING             "K"
#define MODULE_FEATURE_P2P                  "P"
```

### Hardware Type Enum

```cpp
typedef enum {
    HARDWARE_TYPE_UNDEFINED = 0,
    HARDWARE_TYPE_CPU       = 1
} ENUM_HARDWARE_TYPE;
```

---

## Message System

### Routing Logic in `sendJMSG`

<table>
<thead>
<tr><th><code>internal_message</code></th><th><code>targetPartyID</code></th><th>Routing type</th></tr>
</thead>
<tbody>
<tr><td><code>true</code></td><td>any</td><td><code>CMD_TYPE_INTERMODULE</code></td></tr>
<tr><td><code>false</code></td><td><code>""</code></td><td><code>CMD_COMM_GROUP</code></td></tr>
<tr><td><code>false</code></td><td>non-empty</td><td><code>CMD_COMM_INDIVIDUAL</code></td></tr>
</tbody>
</table>

### Wire Protocol Fields

<table>
<thead>
<tr><th>Constant</th><th>JSON key</th><th>Meaning</th></tr>
</thead>
<tbody>
<tr><td><code>INTERMODULE_MODULE_KEY</code></td><td><code>"mk"</code></td><td>Sender module key</td></tr>
<tr><td><code>INTERMODULE_ROUTING_TYPE</code></td><td><code>"ty"</code></td><td>Routing type string</td></tr>
<tr><td><code>ANDRUAV_PROTOCOL_TARGET_ID</code></td><td><code>"tp"</code></td><td>Destination party ID</td></tr>
<tr><td><code>ANDRUAV_PROTOCOL_MESSAGE_TYPE</code></td><td><code>"mt"</code></td><td>Message type integer</td></tr>
<tr><td><code>ANDRUAV_PROTOCOL_MESSAGE_CMD</code></td><td><code>"ms"</code></td><td>Payload object</td></tr>
</tbody>
</table>

### Custom User Message Types

```cpp
#define TYPE_AndruavMessage_USER_RANGE_START  80000
#define TYPE_AndruavMessage_USER_RANGE_END    90000

#define TYPE_MY_SENSOR_DATA  TYPE_AndruavMessage_USER_RANGE_START + 0
#define TYPE_MY_CMD_ACK      TYPE_AndruavMessage_USER_RANGE_START + 1
```

---

## Examples

### 1. Basic Module Communication (`client.cpp`)

```bash
./client MyModule 60000 61111
./client --help
```

```cpp
CModule& cModule = CModule::getInstance();

cModule.defineModule(MODULE_CLASS_GENERIC, "MyModule",
                     generateRandomModuleID(), "0.0.1",
                     Json_de::array());
cModule.addModuleFeatures(MODULE_FEATURE_SENDING_TELEMETRY);
cModule.addModuleFeatures(MODULE_FEATURE_RECEIVING_TELEMETRY);
cModule.setHardware("123456", ENUM_HARDWARE_TYPE::HARDWARE_TYPE_CPU);
cModule.init("0.0.0.0", target_port, "0.0.0.0", listen_port,
             DEFAULT_UDP_DATABUS_PACKET_SIZE);

while (!exit_me) {
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    Json_de message = {{"t", "THIS IS A TEST MESSAGE"}};
    cModule.sendJMSG("", message, TYPE_AndruavMessage_DUMMY, true);
}
```

---

### 2. Binary Data Transmission (`image_sender.cpp`)

```bash
./image_sender ./img.jpeg
```

```cpp
char* content = nullptr;
int content_length = 0;
readBinaryFile(file_name, content, content_length);

Json_de msg_cmd;
msg_cmd["lat"] = 0; msg_cmd["lng"] = 0;
msg_cmd["alt"] = 0; msg_cmd["tim"] = get_time_usec();

// Wire: JSON header + '\0' + image bytes
cModule.sendBMSG("", content, content_length,
                 TYPE_AndruavMessage_IMG, false, msg_cmd);
delete[] content;
```

---

### 3. MAVLink Message Handling (`mavlink_listener.cpp`)

```bash
./mavlink_listener
```

```cpp
#define MESSAGE_FILTER {TYPE_AndruavMessage_MAVLINK, \
                        TYPE_AndruavMessage_SWARM_MAVLINK, ...}

cModule.defineModule(MODULE_CLASS_GENERIC, "mavlink_listener",
                     module_id, "0.0.1",
                     Json_de::array(MESSAGE_FILTER));
cModule.setMessageOnReceive(&onReceive);
cModule.init("0.0.0.0", 60000, "0.0.0.0", 70014,
             DEFAULT_UDP_DATABUS_PACKET_SIZE);

CFacade_Base& facade = CFacade_Base::getInstance();
facade.sendErrorMessage("", 0, ERROR_USER_DEFINED,
                        NOTIFICATION_TYPE_INFO,
                        "Hello World from mavlink_listener");

void onReceive(const char* message, int len, Json_de jMsg) {
    const int msgid = jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE].get<int>();
    // dispatch on msgid...
}
```

---

### 4. Adaptive Rate Control — Sender (`sender_adapter.cpp`)

```bash
./sender_adapter sender_mod 60000 1000
```

```cpp
#define TYPE_CUSTOM_DATA        TYPE_AndruavMessage_USER_RANGE_START + 0
#define TYPE_CUSTOM_CHANGE_RATE TYPE_AndruavMessage_USER_RANGE_START + 1

int delay = 1000; int counter = 0;

void onReceive(const char*, int, Json_de jMsg) {
    if (jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE].get<int>() == TYPE_CUSTOM_CHANGE_RATE) {
        const int delta = jMsg["ms"]["value"].get<int>();
        delay = (delta == 0) ? delay / 2 : delay + delta;
    }
}

while (!exit_me) {
    std::this_thread::sleep_for(std::chrono::milliseconds(delay));
    Json_de msg = {{"t", "DATA"}, {"counter", ++counter}};
    cModule.sendJMSG("", msg, TYPE_CUSTOM_DATA, true);
}
```

---

### 5. Queue-Based Processing — Receiver (`receiver_adapter.cpp`)

```bash
./receiver_adapter receiver_mod 60000 1000
```

```cpp
std::queue<std::vector<char>> messageQueue;
std::mutex queueMutex;
std::condition_variable queueCV;

void onReceive(const char* message, int len, Json_de) {
    std::unique_lock<std::mutex> lock(queueMutex, std::defer_lock);
    if (lock.try_lock()) {
        messageQueue.push(std::vector<char>(message, message + len));
        queueCV.notify_one();
    }
}

void processMessages() {
    while (!messageQueue.empty()) {
        std::this_thread::sleep_for(std::chrono::milliseconds(delay));
        // Backpressure: tell sender to slow down if queue is growing
        Json_de fb = {{"value", (diff > 0 && counter > 2) ? +2*(int)counter : 0}};
        cModule.sendJMSG("sender_mod", fb, TYPE_CUSTOM_CHANGE_RATE, false);
    }
}
```

---

## Advanced Real-World Example: `droneengage_mavlink`

The DroneEngage MAVLink bridge is a production plugin that demonstrates the complete pattern.

### Production Initialization

```cpp
// Load config and generate persistent module key
de::CLocalConfigFile& cLocalConfigFile = de::CLocalConfigFile::getInstance();
cLocalConfigFile.InitConfigFile("de_mavlink.local");
std::string ModuleKey = cLocalConfigFile.getStringField("module_key");
if (ModuleKey.empty()) {
    ModuleKey = std::to_string(get_time_usec());
    cLocalConfigFile.addStringField("module_key", ModuleKey.c_str());
    cLocalConfigFile.apply();
}

cModule.defineModule(MODULE_CLASS_FCB,
                     trim(jsonConfig["module_id"]),
                     ModuleKey, version_string,
                     Json_de::array(MESSAGE_FILTER));  // 30+ message types
cModule.addModuleFeatures(MODULE_FEATURE_SENDING_TELEMETRY);
cModule.addModuleFeatures(MODULE_FEATURE_RECEIVING_TELEMETRY);
cModule.setHardware(hardware_serial, HARDWARE_TYPE_CPU);
cModule.setMessageOnReceive(&onReceive);
cModule.init(jsonConfig["s2s_udp_target_ip"],
             std::stoi(jsonConfig["s2s_udp_target_port"]),
             jsonConfig["s2s_udp_listening_ip"],
             std::stoi(jsonConfig["s2s_udp_listening_port"]),
             udp_chunk_size);
```

### Parser Bridge in `onReceive`

```cpp
void onReceive(const char* message, int len, Json_de jMsg) {
    const int messageType = jMsg[ANDRUAV_PROTOCOL_MESSAGE_TYPE].get<int>();
    if (strcmp(jMsg[INTERMODULE_ROUTING_TYPE].get<std::string>().c_str(),
               CMD_TYPE_INTERMODULE) == 0) {
        if (messageType == TYPE_AndruavModule_ID) {
            cFCBMain.setPartyID(cModule.getPartyId(), cModule.getGroupId());
            return;
        }
    }
    cAndruavResalaParser.parseMessage(jMsg, message, len);  // dispatch to parser
}
```

### Parser Subclass Pattern

```cpp
class CFCBAndruavMessageParser : public de::comm::CAndruavMessageParserBase {
public:
    static CFCBAndruavMessageParser& getInstance() {
        static CFCBAndruavMessageParser instance; return instance;
    }
protected:
    void parseRemoteExecute(Json_de& andruav_message) override;
    void parseCommand(Json_de& andruav_message, const char* full_message,
                      const int& full_message_length,
                      int messageType, uint32_t permission) override;
private:
    de::fcb::CFCBMain&   m_fcbMain    = de::fcb::CFCBMain::getInstance();
    de::fcb::CFCBFacade& m_fcb_facade = de::fcb::CFCBFacade::getInstance();
};
```

### Facade Subclass Pattern

```cpp
class CFCBFacade : public de::comm::CFacade_Base {
public:
    static CFCBFacade& getInstance() {
        static CFCBFacade instance; return instance;
    }
    // Domain-specific send methods (30+):
    void sendHeartBeat(const std::string& target) const;
    void sendGPSInfo(const std::string& target) const;
    void sendMavlinkData(const std::string& target,
                         const mavlink_message_t& msg) const;
    void sendGeoFenceHit(const std::string& target,
                         const std::string fence_name,
                         double distance, bool in_zone) const;
    void requestToFollowLeader(const std::string& target) const;
    // ...
};
```

---

## Port Configuration

<table>
<thead>
<tr><th>Example</th><th>Listen Port</th></tr>
</thead>
<tbody>
<tr><td><code>client.cpp</code></td><td>User-defined (default 61111)</td></tr>
<tr><td><code>image_sender.cpp</code></td><td>50000</td></tr>
<tr><td><code>mavlink_listener.cpp</code></td><td>70014</td></tr>
<tr><td><code>sender_adapter.cpp</code></td><td>70034</td></tr>
<tr><td><code>receiver_adapter.cpp</code></td><td>70024</td></tr>
</tbody>
</table>

Communicator (`de_comm`) listens on port `60000` by default.

---

## Build Flags

<table>
<thead>
<tr><th>Flag</th><th>Effect</th></tr>
</thead>
<tbody>
<tr><td><code>-DDEBUG</code></td><td>Enable basic debug output</td></tr>
<tr><td><code>-DDDEBUG</code></td><td>Verbose: print every sent/received message</td></tr>
<tr><td><code>-DDE_DISABLE_TRY</code></td><td>Disable try-catch in <code>onReceive</code> (performance profiling)</td></tr>
</tbody>
</table>

---

## Thread Safety

- `sendJMSG` and `sendBMSG` hold `std::lock_guard<std::mutex>` on `m_lock`.
- `CUDPClient::sendMSG` uses a separate `m_lock2` mutex for the send socket.
- Never call `sendJMSG` while holding `m_lock` inside a callback (deadlock risk).
- For high-frequency receives, push to a `std::queue` in the callback and process in a separate thread (see `receiver_adapter.cpp`).
- Use `try_lock` in callbacks to avoid blocking the receive thread.

---

## Best Practices

1. **Persistent module key** — Store in `CLocalConfigFile` so the communicator recognizes the module across restarts.
2. **Unique listen port** — Each running module instance must bind a different port.
3. **Minimal message filter** — List only `TYPE_*` values your module handles; reduces overhead.
4. **Delete binary buffers** — `readBinaryFile` allocates with `new[]`; call `delete[]` after `sendBMSG`.
5. **Non-blocking receive callback** — Push to a queue for slow processing; keep the callback fast.
6. **`uninit()` on shutdown** — Always call `cModule.uninit()` before `exit()` to stop UDP threads.
7. **Random or timestamp module key** — Use `generateRandomModuleID()` for testing; use `CLocalConfigFile` for production.

---

## Troubleshooting

<table>
<thead>
<tr><th>Issue</th><th>Solution</th></tr>
</thead>
<tbody>
<tr><td>Module not in WebClient Details</td><td>Verify communicator is on port 60000; check <code>getPartyId()</code> is non-empty</td></tr>
<tr><td><code>Port already in use</code> error</td><td>Choose a different <code>listenPort</code></td></tr>
<tr><td>Messages not received</td><td>Ensure <code>TYPE_*</code> is in your <code>MESSAGE_FILTER</code> array</td></tr>
<tr><td>Segfault in receive callback</td><td>Check <code>jMsg.contains(ANDRUAV_PROTOCOL_MESSAGE_CMD)</code> before accessing</td></tr>
<tr><td>High CPU usage</td><td>Add <code>sleep_for</code> in main loop; avoid busy-wait</td></tr>
<tr><td>Binary message corrupt</td><td>Verify <code>bmsg_length</code> exactly matches the allocated buffer</td></tr>
<tr><td>Module disconnects after restart</td><td>Use persistent <code>module_key</code> from <code>CLocalConfigFile</code></td></tr>
</tbody>
</table>

---

## Additional Resources

- **Overview page** — [./de-custom-plugins.md](./de-custom-plugins.md)
- **Python implementation** — [./de-custom-plugins-python.md](./de-custom-plugins-python.md)
- **Node.js implementation** — [./de-custom-plugins-nodejs.md](./de-custom-plugins-nodejs.md)
- **Core library source** — `droneengage_common/de_databus/`
- **Test examples source** — `droneengage_databus/client/test/`
- **Production example** — `droneengage_mavlink/src/`
- **Message type definitions** — `de_common/de_databus/messages.hpp`
- **Protocol internals** — `droneengage_common/README.md`
