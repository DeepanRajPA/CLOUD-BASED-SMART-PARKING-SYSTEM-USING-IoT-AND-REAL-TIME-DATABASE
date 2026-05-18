# Cloud-Based Smart Parking System Using IoT and Real-Time Database

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Solution Overview](#solution-overview)
3. [Network Architecture & Protocol Stack](#network-architecture--protocol-stack)
4. [Project Scope and Limitations](#project-scope-and-limitations)
5. [System Components](#system-components)
6. [Getting Started](#getting-started)
7. [Technical Details](#technical-details)

---

## Problem Statement

Conventional parking management systems suffer from four fundamental deficiencies:

### 1. No Real-Time Slot Visibility
Drivers approaching a parking facility have no way to know in advance whether spaces are available, leading to wasteful circling behavior and increased emissions.

### 2. No Remote Actuation
Physical barriers separating available from occupied zones typically require either:
- Manual attendants
- Per-slot proximity sensors

Neither solution scales gracefully in multi-level or multi-zone facilities.

### 3. Lack of Distributed Control
Coordinating slot status across geographically distributed nodes requires either:
- Dedicated cabling infrastructure
- Complex wireless network setup

### 4. Non-Standard Communication Interfaces
Existing low-cost IoT prototypes frequently use proprietary communication protocols that do not generalize to production deployment environments.

---

## Solution Overview

This project addresses all four deficiencies through an integrated approach:

- **Vision Pipeline**: Camera-based real-time slot detection for accurate occupancy monitoring
- **Cloud Database**: Firebase Realtime Database as the communication backbone, enabling distributed access and real-time synchronization
- **Distributed Actuation**: Wi-Fi-enabled ESP8266 microcontroller serving as the distributed actuator node
- **Standard Protocols**: All communication over HTTP/HTTPS over TCP/IP, using the same protocol stack as production IoT deployments at scale

---

## Network Architecture & Protocol Stack

This project serves as a practical, end-to-end demonstration of the Computer and Communication Networks curriculum, with every component mapping directly to the OSI and TCP/IP reference models.

### Physical and Data Link Layers
- ESP8266 microcontroller connects to the local network via IEEE 802.11 b/g/n Wi-Fi
- Implements frequency-band negotiation, CSMA/CA medium access control
- Automatic ACK-based frame retransmission

### Network Layer
- Python host and ESP8266 acquire IP addresses from local DHCP server
- Packets routed through NAT gateway to reach Firebase cloud endpoint
- Demonstrates IP addressing, subnet masking, and default gateway operation

### Transport Layer
- Both nodes open TCP connections to Firebase on port 443
- Three-way handshake, flow control, and retransmission handled by OS/SDK TCP stack

### Application Layer
- HTTP/1.1 GET and PUT methods over TLS-encrypted channels (HTTPS)
- JSON-formatted request and response payloads
- DNS resolution of Firebase project endpoint hostname to IP address

### Publish-Subscribe Pattern
- Python node acts as the publisher
- ESP8266 acts as the subscriber
- Firebase Realtime Database acts as the message broker
- Demonstrates contrast between:
  - **Polling**: ESP8266 HTTP GET at 500 ms intervals (latency trade-off)
  - **Event-Driven**: Firebase streaming endpoints / WebSocket upgrade (bandwidth and reliability trade-off)

---

## Project Scope and Limitations

### Scale
- Prototype monitors up to 6 parking slots in a single camera field of view
- Controlled by one ESP8266 node and one servo barrier

### Camera
- Single USB webcam provides the video feed
- System does not handle occlusion between overlapping slots
- Multi-camera stitching not implemented

### Network
- Requires a stable local Wi-Fi network with internet access to reach Firebase
- Fully offline or LAN-only operation not supported

### Latency
- End-to-end latency: 300-800 milliseconds
- Appropriate for parking management
- Not suitable for real-time safety-critical control
- Trade-offs explicitly discussed in project documentation

### Security
- Firebase REST API access secured via HTTPS and API key
- Production deployment would require:
  - Firebase Authentication
  - Security Rules for per-user access control
  - Additional OAuth 2.0 implementation

---

## System Components

### Hardware
- **ESP8266 Microcontroller**: Wi-Fi-enabled IoT device for distributed control
- **USB Webcam**: Video input for parking slot occupancy detection
- **Servo Motor**: Actuates the parking barrier
- **Local Network**: Wi-Fi router with internet connectivity

### Software
- **Python Vision Pipeline**: Real-time slot detection and occupancy analysis
- **Firebase Realtime Database**: Cloud-based data synchronization and storage
- **ESP8266 Firmware**: Firmware handling barrier actuation and database polling
- **HTTP/HTTPS Client Stack**: Standard protocol implementation across all components

---

## Getting Started

### Prerequisites
- Python 3.7 or higher
- ESP8266 board and development environment (Arduino IDE or PlatformIO)
- Firebase project with Realtime Database enabled
- USB webcam
- Local Wi-Fi network with internet access
- Servo motor and mechanical barrier setup

### Installation

1. Clone the repository:
```bash
git clone https://github.com/DeepanRajPA/CLOUD-BASED-SMART-PARKING-SYSTEM-USING-IoT-AND-REAL-TIME-DATABASE.git
cd CLOUD-BASED-SMART-PARKING-SYSTEM-USING-IoT-AND-REAL-TIME-DATABASE
```

2. Set up Python environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. Configure Firebase credentials:
- Update `config.json` with your Firebase project details
- Set your API key and database URL

4. Upload ESP8266 firmware:
- Open the firmware sketch in Arduino IDE / PlatformIO
- Configure Wi-Fi credentials in the firmware
- Upload to your ESP8266 board

5. Run the vision pipeline:
```bash
python main.py
```

---

## Technical Details

### Vision Pipeline
- Real-time occupancy detection using image processing
- Slot status updated to Firebase database
- 500 ms polling interval for consistent updates

### Communication Flow
1. Python vision pipeline detects parking slot status
2. Status published to Firebase Realtime Database via HTTPS
3. ESP8266 polls Firebase at 500 ms intervals
4. Servo barrier actuated based on aggregated slot data

### Database Schema
```json
{
  "parking_slots": {
    "slot_1": {"occupied": true, "timestamp": 1234567890},
    "slot_2": {"occupied": false, "timestamp": 1234567890},
    ...
  },
  "barrier": {
    "state": "open",
    "last_updated": 1234567890
  }
}
```

---

## Performance Metrics

- **Detection Accuracy**: Optimized for 6-slot single camera view
- **Latency**: 300-800 ms end-to-end (detection to barrier actuation)
- **Update Frequency**: 500 ms polling interval on ESP8266
- **Network Protocol**: HTTP/1.1 over TLS 1.2+ (HTTPS)

---

## Future Enhancements

- Multi-camera support with stitching capability
- Increased slot capacity scaling
- Advanced security with Firebase Authentication and Security Rules
- Mobile application for driver notifications
- Machine learning-based occupancy prediction
- Real-time event-driven updates via WebSocket
- Support for multiple barriers and zones

---

## License

This project is provided as an educational demonstration of Computer and Communication Networks principles.

---

## Author

DeepanRajPA

---

## Disclaimer

This system is designed for educational purposes and parking management with 300-800 ms latency tolerance. It is not suitable for real-time safety-critical applications. Production deployment requires additional security hardening, comprehensive testing, and compliance with local regulations.
