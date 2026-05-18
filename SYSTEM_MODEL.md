# Smart Parking System — Model & Methodology

## Problem Definition
Design a distributed system for an N-slot parking facility (N ≤ 6) that:
1. Automatically detects slot occupancy via computer vision (no per-slot sensors)
2. Propagates occupancy to a remote actuator in real time over TCP/IP
3. Controls a barrier and visual indicators without human intervention
4. Maintains end-to-end latency < 2 seconds
5. Maps all components explicitly to OSI reference model layers

## Objectives
- Three-module distributed architecture: Vision → Cloud → Embedded Control
- OpenCV pipeline for real-time video processing and slot classification
- Authenticated HTTPS communication (Firebase REST API + TLS encryption)
- ESP8266 polling system for periodic slot status retrieval
- Servo motor and LED actuation based on parsed JSON data
- Network performance characterization (latency, reliability, throughput)
- OSI/TCP-IP layer mapping and course outcome (CO1, CO3, CO6) alignment
- SDG 9, 11, 12 alignment evaluation

---

## System Architecture

**Three-Tier Distributed Client-Server Model:**

```
Webcam → OpenCV Python → Firebase HTTPS/REST → ESP8266 HTTP GET → Servo + LEDs
(Sensing)    (Processing)     (Cloud Broker)     (Actuation)
```

**Coupling:** Loose coupling via shared Firebase database state; asynchronous, independent tiers.

---

## Module 1: Image Processing (Python + OpenCV)

**Frame Processing Pipeline:**
1. **Frame Acquisition** → VideoCapture at 30 fps (USB webcam)
2. **Grayscale Conversion** → BGR → single-channel via cv2.cvtColor
3. **Gaussian Blur** → 5×5 kernel to suppress noise
4. **ROI Extraction** → Predefined bounding boxes per slot (config file)
5. **Adaptive Thresholding** → cv2.adaptiveThreshold (block 25, offset 16)
6. **Occupancy Classification** → White pixel count vs. calibrated threshold → Occupied/Free
7. **Firebase Write** → HTTPS PUT with JSON payload (requests library)

**Rate Limiting:** Max 1 write/sec per slot to avoid quota overruns.

---

## Module 2: Cloud Communication (Firebase Realtime Database)

**Role:** Central NoSQL JSON database with REST API over HTTPS (port 443).

**Pub-Sub Pattern:** Python = Publisher; ESP8266 = Subscriber; Firebase = Message Store

**Communication Sequence:**
1. Python: HTTP PUT with slot status JSON + API key → Firebase
2. Firebase: Write to database tree → HTTP 200 OK response
3. ESP8266: HTTP GET every 500 ms → Receives current database state as JSON

**Network Events:**
- DNS resolution: `project-id.firebaseio.com` → CDN IP
- TLS 1.2/1.3 handshake & server certificate authentication
- TCP connection management (SYN, SYN-ACK, ACK)

---

## Module 3: Embedded Control (ESP8266 NodeMCU)

**Hardware:** Tensilica L106 80 MHz, 802.11 b/g/n, 80 KB RAM, 4 MB Flash

**Firmware Flow:**
1. **Wi-Fi Connection** → WPA2-PSK authentication, DHCP IP acquisition
2. **HTTP Client Init** → HTTPClient object, Firebase REST endpoint (HTTPS)
3. **Polling Loop** → GET request every 500 ms; parse JSON response (ArduinoJson)
4. **Actuation:**
   - Servo: 0° (barrier open) or 90° (barrier closed) via PWM (GPIO D1)
   - LEDs: Green (available) or Red (full) (GPIO D2, D3)
5. **Full Flag Handling** → If `FULL=true`, close barrier & illuminate red LED only

---

## Data Flow & Alert Logic

**Normal Cycle:**
```
Vision detects change → JSON payload created → HTTPS PUT to Firebase 
→ Firebase stores → ESP8266 GETs (500 ms poll) → Parses JSON → Actuates servo/LEDs
```

**Alert Logic:**
- All N slots occupied → Python sets `FULL: true` → Firebase writes
- ESP8266 checks `FULL` first; if true, closes barrier, red LED only
- Any slot freed → Python clears `FULL` → ESP8266 detects on next poll → Barrier reopens (≤500 ms)

---

## Hardware Components

| Component | Specification | Function |
|-----------|---------------|----------|
| ESP8266 NodeMCU v3 | Tensilica L106 80 MHz, 802.11 b/g/n | Wi-Fi IoT + HTTP polling |
| USB Webcam | 720p/1080p, 30 fps, USB 2.0 | Video frame acquisition |
| Servo Motor (SG90) | 5 V, PWM, 0–180°, 1.8 kg·cm | Barrier actuation |
| LEDs (Red/Green) | 5 mm, 3.3 V, 20 mA | Slot status indicators |
| Ultrasonic Sensor (HC-SR04) | 2–400 cm range, 40 kHz | Proximity detection (backup) |
| Resistors (220 Ω) | ¼ W, ±5% | LED current limiting |

---

## Software Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Host PC | Python 3.x + OpenCV (cv2) | Real-time image processing |
| | requests library | HTTPS REST client |
| | json, time stdlib | Serialization & timing |
| ESP8266 | Arduino IDE v2.x | Firmware development |
| | ESP8266HTTPClient | HTTP operations |
| | ArduinoJson v6 | JSON parsing |
| | Servo library | PWM control |

---

## Protocol Stack & OSI Mapping

| Protocol | OSI Layer | Usage |
|----------|-----------|-------|
| IEEE 802.11 b/g/n | Physical / Data Link | Wireless connectivity; CSMA/CA |
| IPv4 | Network | Packet routing to Firebase via NAT |
| TCP | Transport | Reliable ordered byte-stream (HTTPS) |
| DNS | Application | Domain resolution (`project-id.firebaseio.com` → IP) |
| TLS 1.2/1.3 | Session / Presentation | HTTPS encryption & authentication |
| HTTP/1.1 | Application | GET (ESP8266 read) / PUT (Python write) |
| JSON | Application (Data Format) | Slot status payload serialization |
| Firebase REST API | Application | Pub-Sub messaging via database |

---

## Performance Targets

- **End-to-end latency:** < 2 seconds (vision change → actuator response)
- **ESP8266 poll interval:** 500 ms
- **Firebase write rate:** ≤ 1/sec per slot
- **Measured metrics:** RTT, write time, GET response time, reliability

---

## Alignment

**Course Outcomes:** CO1 (OSI model), CO3 (TCP/IP), CO6 (distributed systems)  
**UN SDGs:** 9 (Industry Innovation), 11 (Sustainable Cities), 12 (Responsible Consumption)
