# RoboticsChallenge MiniMessenger

Classroom board-to-board and board-to-server messaging for Arduino GIGA robot pairs.

## Description

The `MiniMessenger` library simplifies WiFi and MQTT communication for robotics projects. It provides a group-and-board based API, allowing students to easily send messages between specific boards or broadcast to an entire group without managing raw MQTT topics or connection retries.

### Key Features
- **Automatic Connection Management:** Handles WiFi and MQTT connection and reconnection logic.
- **Direct Messaging:** Send data or text to a specific board ID.
- **Group Broadcasting:** Broadcast messages to all boards in the same group.
- **Message Callbacks:** Simple event-driven architecture for handling incoming messages.
- **Lightweight:** Designed to run efficiently on the Arduino GIGA.

## Topic Structure

While the library abstracts topics away, they follow a standardized format compatible with tools like MQTT Explorer:

| Message Type | Topic Pattern |
|--------------|---------------|
| **Status** | `lab/g/{groupId}/board/{boardId}/status` |
| **Direct** | `lab/g/{groupId}/from/{fromId}/to/{targetId}` |
| **Group**  | `lab/g/{groupId}/from/{fromId}/to/all` |

- `{groupId}`: Your assigned group number (e.g., "1").
- `{boardId}`: Your unique board number within the group (e.g., "1" or "2").
- **Status Payloads:** "online" (with retain flag) or "offline" (via LWT).

## Installation

This library requires the **ArduinoMqttClient** library. You can install it via the Arduino Library Manager.

## Bare Minimum Code

To use the library, you need to provide your network credentials and broker details. It is recommended to use a `secrets.h` file.

```cpp
#include <MiniMessenger.h>

// Configuration
const char* ssid = "YOUR_WIFI_SSID";
const char* pass = "YOUR_WIFI_PASSWORD";
const char* broker = "BROKER_IP_OR_HOSTNAME";
const uint16_t port = 1883;
const char* group = "1";
const char* board = "1";

MiniMessenger messenger;

// Callback function for incoming messages
void onMessage(const MessageMetadata& metadata, const uint8_t* payload, size_t length) {
  Serial.print("Message from ");
  Serial.print(metadata.fromBoardId);
  Serial.print(": ");
  
  for (size_t i = 0; i < length; i++) {
    Serial.write(payload[i]);
  }
  Serial.println();
}

void setup() {
  Serial.begin(115200);
  
  // Initialize messenger
  messenger.onMessage(onMessage);
  messenger.begin(ssid, pass, broker, port, group, board);
}

void loop() {
  // Keep connections alive and poll for messages
  messenger.loop();

  // Example: Send a message every 5 seconds
  static unsigned long lastSend = 0;
  if (millis() - lastSend > 5000) {
    messenger.sendToGroup("Hello World!");
    lastSend = millis();
  }
}
```

## API Reference

### Initialization
- `bool begin(ssid, password, brokerHost, brokerPort, groupId, boardId)`: Connects to WiFi and MQTT.
- `void loop()`: Must be called in the main `loop()` to process messages and maintain connections.

### Sending Messages
- `bool sendToBoard(targetBoardId, text)`: Send a string to a specific board.
- `bool sendToGroup(text)`: Broadcast a string to everyone in your group.
- `bool sendToBoard(targetBoardId, data, length)`: Send raw bytes to a specific board.

### Receiving Messages
- `void onMessage(callback)`: Register a function to handle incoming messages. The callback receives `MessageMetadata` (containing `fromBoardId`, `groupId`, etc.) and the payload.

### Connection Status
- `bool isConnected()`: Returns `true` if both WiFi and MQTT are connected.

### Contributing and Linting

To ensure this library remains compatible with the official Arduino Library Manager, this repository uses **Arduino Lint** via GitHub Actions. 

Whenever you push code or open a Pull Request, our automated workflow will check the repository structure, `library.properties` file, and code styling to ensure it meets strict Arduino Registry compliance. Please ensure the Arduino Lint badge is passing (green) before requesting a merge!
