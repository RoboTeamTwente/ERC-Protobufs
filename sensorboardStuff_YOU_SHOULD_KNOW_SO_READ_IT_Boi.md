# Sensor Board

Documentation for all `.proto` definitions used in the sensor board system.

---

## sensor.proto

Defines shared core types used across all sensor modules.

### Purpose

Provides a common `SensorState` enum to standardize operational states across all sensors in the system.

### Contents

-`SensorState`

  -`SENSOR_IDLE`

  -`SENSOR_OPERATING`

  -`SENSOR_CALIBRATING`

  -`SENSOR_ERROR`

### Role

Acts as a foundational dependency for all sensor-specific `.proto` files to ensure consistent state representation.

---

## ph_sensor.proto

Defines the data structure and error handling for the pH sensor.

### Purpose

Describes measurements, operational state and error conditions for the pH sensor module.

### Contents

-`PHSensorInformation`

  -`ph_value` (0–14 scale)

  -`voltage` (mV)

  -`temperature` (°C)

  -`state` (`SensorState`)

  -`error_code` (`PHErrorCode`)

-`PHErrorCode`

- Communication, calibration, probe and temperature errors

### Role

Represents chemical measurement data and integrates with board-level diagnostics.

---

## imu_sensor.proto

Defines the data structure and error handling for the IMU (Inertial Measurement Unit).

### Purpose

Represents motion and orientation data from a 9-axis IMU sensor.

### Contents

-`IMUSensorInformation`

- Accelerometer data (x, y, z)
- Gyroscope data (x, y, z)
- Magnetometer data (x, y, z)
- Calibration status

  -`state` (`SensorState`)

  -`error_code` (`IMUErrorCode`)

-`IMUErrorCode`

- Communication, calibration, gyroscope, accelerometer and magnetometer errors

### Role

Provides motion tracking and orientation data for navigation and system monitoring.

---

## gps_sensor.proto

Defines the data structure and error handling for the GPS sensor.

### Purpose

Represents positioning, velocity and fix quality data from a GPS module.

### Contents

-`GPSSensorInformation`

- Latitude, longitude, altitude
- Speed and heading
- HDOP and VDOP precision metrics
- Satellite count
- Fix quality (`GPSFixQuality`)

  -`state` (`SensorState`)

  -`error_code` (`GPSErrorCode`)
- UTC timestamp (ms)

-`GPSFixQuality`

- No fix, GPS, DGPS, RTK, RTK Float, PPS

-`GPSErrorCode`

- Communication, antenna, signal quality, invalid data

### Role

Provides global positioning data and integrates into system-level diagnostics.

---

## diagnostics.proto

Defines the board-level diagnostics message aggregating all sensors.

### Purpose

Aggregates sensor data and reports overall sensor board health.

### Contents

-`SensorBoardDiagnostics`

- Board-level state

  -`PHSensorInformation`

  -`IMUSensorInformation`

  -`GPSSensorInformation`
- Board temperature
- Board voltage

### Role

Serves as a centralized health and telemetry snapshot of the entire sensor board.

---

## sensor_board.proto

Defines the message envelope and handshake protocol for board communication.

### Purpose

Wraps serialized protobuf messages and defines board-level message type negotiation.

### Contents

-`PBMessageEnvelope`

  -`identifier` (board ID + message identifier)

  -`data` (serialized protobuf payload)

-`PBBroadcastSendingTypes`

- Declares supported outgoing message identifiers

-`PBBroadcastingReceivingTypes`

- Declares supported incoming message identifiers

### Role

Provides transport-level encapsulation and capability discovery between sensor boards.
