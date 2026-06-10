# Low-Power IoT Communication: From Protocol Choice to System-Level Energy Design

## How IoT Systems Stay Connected While Minimizing Energy Consumption

> Course: Computer Networks
> Team: Group 5
> Topic: Low-Power IoT Communication
> Student: Gyuri Park
> Student ID: 2024270637
> Presentation Video: [Watch Here](PASTE_VIDEO_LINK_HERE)

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Why Energy Efficiency Matters in IoT](#2-why-energy-efficiency-matters-in-iot)
3. [Low-Power Design Principles](#3-low-power-design-principles)
4. [Communication Scheduling: Duty Cycling and Event-Driven Transmission](#4-communication-scheduling-duty-cycling-and-event-driven-transmission)
5. [Protocol-Level Trade-offs](#5-protocol-level-trade-offs)
6. [System-Level Optimization: Reducing Data Movement](#6-system-level-optimization-reducing-data-movement)
7. [Hardware Support for Low-Power IoT](#7-hardware-support-for-low-power-iot)
8. [Protocol Selection Strategy](#8-protocol-selection-strategy)
9. [Logical Deductions](#9-logical-deductions)
10. [Conclusion](#10-conclusion)
11. [References](#11-references)

---

## 1. Problem Statement

IoT devices are usually small, battery-powered, and widely distributed. Unlike ordinary computers, many IoT devices are expected to operate for months or years without frequent charging or maintenance.

Examples include:

| IoT Device         | Main Challenge                        |
| ------------------ | ------------------------------------- |
| Temperature sensor | Periodic sensing with limited battery |
| Wearable device    | Small battery and frequent updates    |
| Smart meter        | Long-term field operation             |
| Industrial sensor  | Difficult maintenance and replacement |

The main problem is:

> **How can an IoT system stay connected while using as little energy as possible?**

This question is important because communication, computation, and data movement all consume energy. Therefore, low-power IoT design should not be treated as only a battery problem. It should be understood as a system-level design problem.

---

## 2. Why Energy Efficiency Matters in IoT

Energy consumption in IoT systems mainly comes from three areas.

### 2.1 Wireless Communication

Wireless communication is often one of the largest sources of energy consumption. Energy is spent when the device wakes up the radio, transmits packets, waits for acknowledgements, or retransmits lost packets.

For example, if a temperature sensor sends a packet too frequently, even small messages can quickly drain the battery.

### 2.2 CPU Computation

The CPU consumes energy when it processes sensor data, performs encryption, executes control logic, or manages application-level tasks.

Even if the communication protocol is lightweight, unnecessary computation can still increase power consumption.

### 2.3 Data Movement

Data movement is another hidden energy cost. When data is repeatedly copied between CPU, memory, buffers, and storage, the device spends additional energy.

This means that reducing packet size is not enough. A low-power IoT system should also reduce unnecessary internal data movement.

---

## 3. Low-Power Design Principles

The goal of low-power IoT communication is not to remove communication completely. The real goal is to communicate only when communication is useful.

The core principle can be summarized as follows:

```text
Send less data → Sleep more often → Move less data
```

Each part has a specific meaning.

| Principle        | Meaning                                          | Energy Benefit                   |
| ---------------- | ------------------------------------------------ | -------------------------------- |
| Send less data   | Reduce unnecessary packets and protocol overhead | Less radio activity              |
| Sleep more often | Keep radio and CPU inactive when possible        | Lower active power consumption   |
| Move less data   | Reduce CPU-memory traffic and buffer copies      | Lower internal processing energy |

This principle leads to an important design rule:

> **The communication method should be selected based on the importance of the data.**

For example, a routine temperature reading is repeated and loss-tolerant, so it can use lightweight communication. However, a fire alarm or control command is critical, so it should use reliable communication even if it requires more overhead.

---

## 4. Communication Scheduling: Duty Cycling and Event-Driven Transmission

### 4.1 Duty Cycling

Duty cycling is a low-power technique where an IoT device alternates between active and sleep states.

For example:

```text
Active for 1 second → Sleep for 9 seconds → Active for 1 second → Sleep for 9 seconds
```

In this case, the device is active only 10% of the time and sleeps 90% of the time.

This reduces energy consumption because the radio and CPU do not remain active continuously.

### 4.2 Periodic Communication

In periodic communication, the device sends data at fixed intervals.

Example:

```text
A temperature sensor sends data every 10 seconds.
```

This approach is simple and predictable. However, it may waste energy if the sensor sends data even when the measured value has not changed.

### 4.3 Event-Driven Communication

In event-driven communication, the device stays quiet until a meaningful event occurs.

Example:

```text
A fire sensor sends data only when abnormal smoke or temperature is detected.
```

This can save more energy than periodic communication because the device avoids unnecessary transmissions.

### 4.4 Trade-off

Duty cycling and event-driven communication reduce energy consumption, but they can introduce trade-offs.

| Benefit                  | Trade-off                         |
| ------------------------ | --------------------------------- |
| Lower energy consumption | Higher latency                    |
| Longer battery life      | Lower monitoring resolution       |
| Less network traffic     | Possible delay in event detection |

Therefore, low-power design requires balancing energy savings with application requirements.

---

## 5. Protocol-Level Trade-offs

There is no single best protocol for all IoT systems. The best choice depends on reliability, latency, energy, topology, range, and security requirements.

### 5.1 TCP vs UDP

TCP and UDP show a clear reliability-energy trade-off.

| Feature     | TCP                                          | UDP                   |
| ----------- | -------------------------------------------- | --------------------- |
| Connection  | Connection-oriented                          | Connectionless        |
| Reliability | High reliability with ACK and retransmission | No delivery guarantee |
| Overhead    | Higher                                       | Lower                 |
| Latency     | Higher                                       | Lower                 |
| Energy cost | Higher                                       | Lower                 |

TCP is suitable for critical data because it provides reliable delivery. However, this reliability requires additional overhead.

UDP is suitable for routine sensor data because it is lightweight and has lower latency. However, UDP does not guarantee delivery or ordering.

The key question is not simply:

```text
Should we use TCP or UDP?
```

The better question is:

```text
Which data requires reliability, and which data can tolerate loss?
```

### 5.2 MQTT

MQTT is a lightweight publish/subscribe protocol commonly used in IoT telemetry.

Basic structure:

```text
Sensor / Publisher → MQTT Broker → Subscriber / Cloud Application
```

MQTT is useful because publishers and subscribers do not need to communicate directly. Instead, the broker handles message routing.

MQTT also provides QoS levels.

| QoS Level | Meaning       | Overhead |
| --------- | ------------- | -------- |
| QoS 0     | At most once  | Lowest   |
| QoS 1     | At least once | Medium   |
| QoS 2     | Exactly once  | Highest  |

MQTT is suitable for cloud-connected sensor reporting, especially when data is sent repeatedly.

### 5.3 CoAP

CoAP is designed for constrained IoT environments. It runs over UDP and adds optional reliability at the application layer.

CoAP supports two important message types:

| Message Type | Meaning                     | Suitable Case                     |
| ------------ | --------------------------- | --------------------------------- |
| NON          | No acknowledgement required | Routine sensor reading            |
| CON          | Acknowledgement required    | Critical alarm or control command |

The important idea is selective reliability.

> **Do not pay the reliability cost for every packet. Use reliability only when the data requires it.**

This makes CoAP useful in low-power IoT systems because it keeps the lightweight nature of UDP while allowing reliability when needed.

### 5.4 BLE and Zigbee

Transport and application protocols are not the only factors. Wireless access technology also affects energy consumption.

| Technology | Strength                                 | Suitable Example                |
| ---------- | ---------------------------------------- | ------------------------------- |
| BLE        | Very low-power short-range communication | Heart-rate sensor to smartphone |
| Zigbee     | Low-power mesh networking                | Smart home room sensors         |

BLE is suitable for nearby personal-area communication. Zigbee is suitable when many low-power nodes need local mesh connectivity.

---

## 6. System-Level Optimization: Reducing Data Movement

Even if wireless communication is optimized, internal data movement can still waste energy.

In a conventional system, data often moves repeatedly between CPU and memory.

```text
CPU ↔ Memory ↔ Buffer ↔ Storage
```

This repeated movement creates delay and energy overhead.

One possible solution is PIM, or Processing-In-Memory. PIM moves computation closer to where the data is stored.

```text
Memory + Near-Data Computing
```

This can reduce unnecessary CPU-memory traffic.

For IoT devices, this is important because many systems process small sensor values repeatedly. Reducing internal movement can lower both energy consumption and processing delay.

---

## 7. Hardware Support for Low-Power IoT

Protocol selection reduces communication overhead, but hardware-level techniques are also necessary for low-power design.

### 7.1 Clock Gating

Clock gating stops clock signals to unused circuit blocks.

This reduces dynamic power consumption because inactive blocks do not continue switching unnecessarily.

### 7.2 Power Gating

Power gating turns off unused circuit blocks completely.

This reduces leakage power, which is important when a device spends a long time in sleep mode.

### 7.3 PIM / Near-Memory Computing

PIM reduces data movement by processing data closer to memory.

This helps reduce CPU-memory traffic, delay, and energy consumption.

The layered structure of low-power IoT design can be summarized as follows:

```text
Application Layer → Transport Layer → Wireless Layer → Hardware Layer
MQTT / CoAP       → TCP / UDP       → BLE / Zigbee   → Gating / PIM
```

This shows that low-power IoT communication is not solved by one layer alone. Communication, computation, wireless access, and hardware must work together.

---

## 8. Protocol Selection Strategy

A practical IoT system should select protocols based on the type and importance of data.

| Scenario                | Example Data                    | Primary Requirement      | Suitable Choice  |
| ----------------------- | ------------------------------- | ------------------------ | ---------------- |
| Periodic telemetry      | Temperature / humidity          | Low overhead             | MQTT or CoAP NON |
| Critical alarm          | Fire, smoke, intrusion          | Reliability              | TCP or CoAP CON  |
| Nearby wearable sensor  | Heart-rate, motion              | Very low power           | BLE              |
| Many local sensor nodes | Room sensors connected to hub   | Mesh scalability         | Zigbee           |
| Security-sensitive data | Health data or control commands | Protection and integrity | TLS / DTLS       |

The main conclusion is:

> **Low-power design is a selection strategy, not a single protocol choice.**

---

## 9. Logical Deductions

Based on the analysis, several logical deductions can be made.

### Deduction 1: Reliability should be proportional to data importance.

If every packet uses the highest reliability level, the system consumes unnecessary energy.
However, if critical data uses an unreliable method, the system may fail in dangerous situations.

Therefore, routine data should use lightweight transmission, while critical data should use reliable transmission.

### Deduction 2: Sleeping is often more effective than optimizing packet size alone.

Reducing packet size helps, but if the device wakes up too frequently, energy is still wasted.
Duty cycling and event-driven communication reduce active time, which directly improves battery life.

### Deduction 3: Protocol optimization alone is not enough.

Even if the network protocol is lightweight, CPU computation and data movement can still waste energy.
Therefore, hardware techniques such as clock gating, power gating, and PIM should be considered together.

### Deduction 4: The best protocol depends on the application scenario.

A wearable heart-rate sensor, a fire alarm, and a smart home sensor network have different requirements.
Therefore, the protocol should be selected according to reliability, latency, energy, topology, range, and security.

---

## 10. Conclusion

Low-power IoT communication is about matching energy cost to data value.

The most important design principles are:

1. Communicate only when useful.
2. Select protocols according to data importance.
3. Reduce unnecessary packet transmission and internal data movement.
4. Use hardware-level power-saving techniques when possible.
5. Optimize the entire system, not only the network protocol.

Final message:

> **Send only the necessary data, at the necessary time, using the most efficient communication path.**

---

## 11. References

1. IETF, RFC 768: User Datagram Protocol.
   https://datatracker.ietf.org/doc/html/rfc768

2. IETF, RFC 9293: Transmission Control Protocol.
   https://datatracker.ietf.org/doc/html/rfc9293

3. IETF, RFC 7252: The Constrained Application Protocol.
   https://datatracker.ietf.org/doc/html/rfc7252

4. OASIS, MQTT Version 5.0 Standard.
   https://www.oasis-open.org/standard/mqtt-v5-0-os/

5. Bluetooth SIG, Bluetooth Low Energy Documentation.
   https://www.bluetooth.com/bluetooth-resources/

6. Connectivity Standards Alliance, Zigbee Technology.
   https://csa-iot.org/all-solutions/zigbee/

7. Group 5 Computer Networks Presentation, “Low-Power IoT Communication,” 2026.

---

Computer Networks · Group 5 · Low-Power IoT Communication
