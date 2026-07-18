# Unified Diagnostic Services (UDS) – ISO 14229 Overview

# 1. What is UDS?

**Unified Diagnostic Services (UDS)** is an application-layer diagnostic communication protocol standardized in **ISO 14229**. It enables diagnostic testers (service tools) to communicate with Electronic Control Units (ECUs) for fault diagnosis, software updates, configuration, security access, and maintenance.

UDS is commonly used over:

- CAN (ISO 15765-3 / ISO-TP)
- CAN FD
- FlexRay
- Ethernet (DoIP – ISO 13400)
- LIN (limited implementations)

---

# 2. Why is UDS Needed?

Modern vehicles contain dozens of ECUs controlling powertrain, body electronics, ADAS, infotainment, and safety systems.

UDS provides:

✅ Standardized diagnostics across OEMs and suppliers

✅ ECU fault detection and reporting

✅ Software flashing and reprogramming

✅ Manufacturing and end-of-line testing

✅ Service workshop diagnostics

✅ Security-controlled ECU access

✅ Configuration and calibration support

✅ Regulatory compliance and maintenance support

---

# 3. UDS Communication Architecture

```text
+-----------------------+
| Diagnostic Tester     |
| (Scan Tool)           |
+-----------+-----------+
            |
            | UDS Request
            v
+-----------------------+
| Vehicle Network       |
| CAN / CAN FD / DoIP   |
+-----------+-----------+
            |
            v
+-----------------------+
| ECU                   |
| Engine/ABS/BCM/etc.   |
+-----------------------+
            |
            +---- Positive Response
            |
            +---- Negative Response
```

---

# 4. UDS Message Structure

## Request

```text
+------+------------------+
| SID  | Parameters       |
+------+------------------+
```

## Positive Response

```text
+---------+---------------+
| SID+0x40| Parameters    |
+---------+---------------+
```

Example:

```text
Request : 10 03
Response: 50 03
```

## Negative Response

```text
7F <Requested SID> <NRC>
```

Example:

```text
7F 10 22
```

---

# 5. Diagnostic Sessions

```text
                 +----------------+
                 | Default (0x01) |
                 +--------+-------+
                          |
        +-----------------+----------------+
        |                                  |
        v                                  v
+---------------+                 +---------------+
| Programming   |                 | Extended      |
| Session 0x02  |                 | Session 0x03  |
+---------------+                 +---------------+
        |
        v
+---------------+
| Safety System |
| Session 0x04  |
+---------------+
```

---

# 6. Common UDS Services (ISO 14229)

| SID | Service Name |
|------|-------------|
| 0x10 | Diagnostic Session Control |
| 0x11 | ECU Reset |
| 0x14 | Clear Diagnostic Information |
| 0x19 | Read DTC Information |
| 0x22 | Read Data By Identifier |
| 0x23 | Read Memory By Address |
| 0x24 | Read Scaling Data By Identifier |
| 0x27 | Security Access |
| 0x28 | Communication Control |
| 0x29 | Authentication |
| 0x2A | Read Data By Periodic Identifier |
| 0x2C | Dynamically Define Data Identifier |
| 0x2E | Write Data By Identifier |
| 0x2F | Input Output Control By Identifier |
| 0x31 | Routine Control |
| 0x34 | Request Download |
| 0x35 | Request Upload |
| 0x36 | Transfer Data |
| 0x37 | Request Transfer Exit |
| 0x3D | Write Memory By Address |
| 0x3E | Tester Present |
| 0x85 | Control DTC Setting |
| 0x87 | Link Control |

---

# 7. Important UDS Services and Subfunctions

## 7.1 Diagnostic Session Control (0x10)

| Subfunction | Description |
|------------|-------------|
| 0x01 | Default Session |
| 0x02 | Programming Session |
| 0x03 | Extended Diagnostic Session |
| 0x04 | Safety System Diagnostic Session |

Example:

```text
Request : 10 03
Response: 50 03
```

---

## 7.2 ECU Reset (0x11)

| Subfunction | Description |
|------------|-------------|
| 0x01 | Hard Reset |
| 0x02 | Key Off-On Reset |
| 0x03 | Soft Reset |
| 0x04 | Enable Rapid Power Shutdown |
| 0x05 | Disable Rapid Power Shutdown |

---

## 7.3 Security Access (0x27)

```text
Tester                ECU
   |---- Request Seed ---->|
   |<---- Seed ------------|
   |---- Send Key -------->|
   |<---- Access Granted --|
```

| Subfunction | Meaning |
|------------|---------|
| 0x01 | Request Seed Level 1 |
| 0x02 | Send Key Level 1 |
| 0x03 | Request Seed Level 2 |
| 0x04 | Send Key Level 2 |

---

## 7.4 Read Data By Identifier (0x22)

Used to read ECU parameters.

Examples:

| DID | Description |
|------|------------|
| F190 | VIN |
| F187 | Vehicle Manufacturer Spare Part Number |
| F18C | ECU Serial Number |

Example:

```text
22 F1 90
```

---

## 7.5 Write Data By Identifier (0x2E)

Used for coding/configuration.

Example:

```text
2E F1 90 xx xx
```

---

## 7.6 Routine Control (0x31)

| Subfunction | Description |
|------------|-------------|
| 0x01 | Start Routine |
| 0x02 | Stop Routine |
| 0x03 | Request Routine Results |

Typical Uses:

- Erase Flash
- Integrity Check
- Calibration
- Self-Test

---

## 7.7 Read DTC Information (0x19)

| Subfunction | Description |
|------------|-------------|
| 0x01 | Number Of DTCs |
| 0x02 | Read DTC By Status Mask |
| 0x04 | Snapshot Record |
| 0x06 | Extended Data Record |
| 0x0A | Supported DTCs |

---

## 7.8 Communication Control (0x28)

| Subfunction | Description |
|------------|-------------|
| 0x00 | Enable Rx and Tx |
| 0x01 | Enable Rx Disable Tx |
| 0x02 | Disable Rx Enable Tx |
| 0x03 | Disable Rx and Tx |

---

## 7.9 Tester Present (0x3E)

Keeps diagnostic session alive.

```text
Request : 3E 00
Response: 7E 00
```

---

## 7.10 Control DTC Setting (0x85)

| Subfunction | Description |
|------------|-------------|
| 0x01 | ON |
| 0x02 | OFF |

---

# 8. Negative Response Codes (NRC)

Negative Response Format:

```text
7F <SID> <NRC>
```

## Common NRCs

| NRC | Meaning |
|------|---------|
| 0x10 | General Reject |
| 0x11 | Service Not Supported |
| 0x12 | Subfunction Not Supported |
| 0x13 | Incorrect Message Length Or Format |
| 0x14 | Response Too Long |
| 0x21 | Busy Repeat Request |
| 0x22 | Conditions Not Correct |
| 0x24 | Request Sequence Error |
| 0x25 | No Response From Subnet Component |
| 0x26 | Failure Prevents Execution |
| 0x31 | Request Out Of Range |
| 0x33 | Security Access Denied |
| 0x35 | Invalid Key |
| 0x36 | Exceeded Number Of Attempts |
| 0x37 | Required Time Delay Not Expired |
| 0x70 | Upload Download Not Accepted |
| 0x71 | Transfer Data Suspended |
| 0x72 | General Programming Failure |
| 0x73 | Wrong Block Sequence Counter |
| 0x78 | Response Pending |
| 0x7E | Subfunction Not Supported In Active Session |
| 0x7F | Service Not Supported In Active Session |

---

# 9. Flash Programming Flow

```text
Diagnostic Session Control (0x10)
            |
            v
Security Access (0x27)
            |
            v
Request Download (0x34)
            |
            v
Transfer Data (0x36)
            |
            v
Request Transfer Exit (0x37)
            |
            v
ECU Reset (0x11)
```

---

# 10. Typical UDS Diagnostic Workflow

```text
Tester
  |
  |---- Diagnostic Session Control
  |
  |---- Security Access
  |
  |---- Read DID
  |
  |---- Read DTC
  |
  |---- Routine Control
  |
  |---- Flash Programming
  |
  |---- ECU Reset
  |
  v
 ECU
```

---

# 11. ISO 14229 Standards Family

| Standard | Description |
|-----------|-------------|
| ISO 14229-1 | Application Layer Requirements |
| ISO 14229-2 | Session Layer Services |
| ISO 14229-3 | UDS on CAN |
| ISO 14229-4 | UDS on FlexRay |
| ISO 14229-5 | UDS on IP (DoIP) |
| ISO 14229-6 | UDS on K-Line |

---

# 12. Key Takeaways

- UDS is the industry-standard diagnostic protocol for automotive ECUs.
- Defined by ISO 14229.
- Provides diagnostics, configuration, security, and reprogramming capabilities.
- Uses Services (SID), Subfunctions, DIDs, DTCs, and NRCs.
- Widely used across OEMs, suppliers, workshops, and manufacturing plants.
- Essential for ECU flashing, fault management, and vehicle maintenance.

