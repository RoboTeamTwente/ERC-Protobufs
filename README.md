The official protobuffer packets for communication between Embedded, Software & Control.
This repository can be implemented as a submodule into other repositories. It is in ERC-Embedded.

It is extremely important that each protobuffer has a UNIQUE name, irrespective of the folder it resides in.
This is because when you import another protobuf, you use "import [name].proto"

For more information on submodules, see: https://git-scm.com/book/en/v2/Git-Tools-Submodules

After cloning a repo with submodules, run:
git submodule init
git submodule update

or clone the repo with: git clone --recurse-submodules [repo_link]

## How `envelope.proto` Works
[Link to source: `ERC-Protobufs/components/common/envelope.proto`](ERC-Protobufs/components/common/envelope.proto)

---

The Protocol Buffer definitions are organized into a modular hierarchy to separate shared logic from board-specific data:

* **`components/common/`**: Shared definitions used across the entire system.
    * `envolop.proto`: Core message wrapper, BoardId, and SystemMessageType enums.
    * `motor.proto`: Standardized motor telemetry and control structures.
    * `sensor.proto`: Common sensor state enums.
* **`components/[board_name]/`**: Board-specific payloads and message type enums (point of contact for info)
    * **arm_board/**: Lisa - includes `message_types.proto` (ArmBoardMessageType enum)
    * **driving_board/**: İrem - includes `message_types.proto` (DrivingBoardMessageType enum)
    * **sensorboard/**: Shishir - includes `message_types.proto` (SensorBoardMessageType enum)
    * **debugging_board/**: To be added by Nick
    * **surface_sampler_board/**: To be created

---

### Board Identifiers
Each board is assigned a unique identifier in the high byte of the MessageType enum:

| ID (Hex) | Board Name | Purpose | Message Range |
| :--- | :--- | :--- | :--- |
| **0x00** | `BOARD_RESERVED` | System messages / Error state | 0x00XX |
| **0x01** | `BOARD_ARM` | Robotic arm control and feedback | 0x01XX |
| **0x02** | `BOARD_DEBUGGING` | External telemetry and logging | 0x02XX |
| **0x03** | `BOARD_DRIVING` | Driving, movement motors and feedback | 0x03XX |
| **0x04** | `BOARD_SENSOR` | Sensor data and feedback | 0x04XX |
| **0x05** | `BOARD_SURFACE_SAMPLER` | Surface sampling operations | 0x05XX |

---

### Message Envelope
All board-specific messages are wrapped in `PBMessageEnvelope` for unified communication.

#### Message Type System
Instead of a single monolithic enum, message types are now defined modularly in each board's folder:

- **`SystemMessageType`** (in `envolop.proto`): System-level messages (0x00XX range)
- **`ArmBoardMessageType`** (in `arm_board/message_types.proto`): Arm board messages (0x01XX range)
- **`DrivingBoardMessageType`** (in `driving_board/message_types.proto`): Driving board messages (0x03XX range)
- **`SensorBoardMessageType`** (in `sensorboard/message_types.proto`): Sensor board messages (0x04XX range)

**Format:** `0xBBMM`
- **BB** (high byte): Board identifier (0x00-0xFF)
- **MM** (low byte): Specific message type (0x00-0xFF)

#### Sensor Board Message Types (0x04XX)
Defined in `components/sensorboard/message_types.proto`:
* `MSG_SENSOR_GPS_INFO (0x0400)`: GPS coordinates, speed, satellites
* `MSG_SENSOR_IMU_INFO (0x0401)`: Accelerometer, gyroscope, magnetometer
* `MSG_SENSOR_PH_INFO (0x0402)`: pH value, voltage, temperature
* `MSG_SENSOR_DIAGNOSTICS (0x0403)`: Combined sensor health

#### System Messages (0x00XX)
Defined in `components/common/envolop.proto`:
* `MSG_BROADCAST_SENDING_TYPES (0x0001)`: Handshake - advertises what messages a node sends
* `MSG_BROADCAST_RECEIVING_TYPES (0x0002)`: Handshake - advertises what messages a node receives

#### Data Payload
* **`data (bytes)`**: Dynamically-sized byte array containing the serialized board-specific protobuf message.

---

### Usage Example - Sensor Board
```cpp
// Import the sensor board message types
#include "sensorboard/message_types.pb.h"
#include "sensorboard/gps_sensor.pb.h"
#include "common/envolop.pb.h"

// Creating and sending GPS data from Sensor Board

// Step 1: Create and populate sensor message
GPSSensorInformation gps_data;
gps_data.set_latitude(43.6532);
gps_data.set_longitude(-79.3832);
gps_data.set_satellites(12);
// ... set other fields

// Step 2: Serialize to bytes
std::string serialized_gps;
gps_data.SerializeToString(&serialized_gps);

// Step 3: Create envelope with SensorBoardMessageType enum
PBMessageEnvelope envelope;
envelope.set_message_type(MSG_SENSOR_GPS_INFO);  // From SensorBoardMessageType enum (0x0400)
envelope.set_data(serialized_gps);

// Step 4: Serialize and send
std::string packet;
envelope.SerializeToString(&packet);
sendto(socket_fd, packet.data(), packet.size(), 0, &dest_addr, sizeof(dest_addr));

// ===== RECEIVING SIDE =====

// Step 5: Receive and deserialize envelope
PBMessageEnvelope received_envelope;
received_envelope.ParseFromArray(buffer, bytes_received);

// Step 6: Extract board ID from message_type
int32_t msg_type = received_envelope.message_type();
uint8_t board_id = (msg_type >> 8) & 0xFF;  // Extract high byte

// Step 7: Route based on board and message type
if (board_id == BOARD_SENSOR) {
    switch(msg_type) {
        case MSG_SENSOR_GPS_INFO: {
            GPSSensorInformation gps;
            gps.ParseFromString(received_envelope.data());
            // Use gps.latitude(), gps.longitude(), etc.
            break;
        }
        case MSG_SENSOR_IMU_INFO: {
            IMUSensorInformation imu;
            imu.ParseFromString(received_envelope.data());
            // Use imu.accel_x(), imu.gyro_y(), etc.
            break;
        }
        // ... handle other sensor messages
    }
}
```
# GOD BLESS☮️