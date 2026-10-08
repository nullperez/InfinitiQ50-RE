# Infiniti Q50 CAN Bus Decoding

This is an ongoing effort to reverse engineer the CAN bus on my Infiniti Q50, decoding as many messages and signals as possible. Everything here was worked out empirically: logging traffic, triggering things in the car (doors, lights, pedals, buttons), and watching which bytes change.

> [!WARNING]
> **This is unofficial and incomplete.** None of it comes from Nissan/Infiniti documentation. Some signals are educated guesses, some bit widths and scaling factors are approximate, and some of it may simply be wrong. Verify anything before relying on it, and **don't use this data for anything safety-critical.**

## How to read this

- Each message has a header (CAN ID, description, DLC, interval) and a byte/bit grid. Bit 7 is the most significant bit of each byte.
- Signals that span multiple bytes are merged across the grid. Where a signal wraps into part of the next byte, the continuation is marked with ↑ or ↓.
- A `?` after a name means the meaning is unconfirmed.
- Blank cells haven't been decoded yet. They may be unused, or I just haven't figured them out.
- Enum tables list the observed values for multi-bit signals. Values missing from a table haven't been seen.

Corrections and additions are welcome. If you've confirmed (or disproved) anything here on your own car, please open an issue or PR.

- [0x002 Steering Angle](#0x002-steering-angle)
- [0x160 Accelerator Pedal](#0x160-accelerator-pedal)
- [0x180 Engine RPM](#0x180-engine-rpm)
- [0x182](#0x182)
- [0x1A5 AWD](#0x1a5-awd)
- [0x216 BCM](#0x216-bcm)
- [0x280 Cluster Status #1](#0x280-cluster-status-1)
- [0x284 Front Wheel Speeds](#0x284-front-wheel-speeds)
- [0x285 Rear Wheel Speeds](#0x285-rear-wheel-speeds)
- [0x292 Forces](#0x292-forces)
- [0x2B1 ADAS](#0x2b1-adas)
- [0x2DE Cluster Status #2](#0x2de-cluster-status-2)
- [0x351 Intelligent Key](#0x351-intelligent-key)
- [0x354 Chassis Control](#0x354-chassis-control)
- [0x355 Cluster Status #3](#0x355-cluster-status-3)
- [0x358](#0x358)
- [0x35D](#0x35d)
- [0x385 TPMS Info](#0x385-tpms-info)
- [0x3AF](#0x3af)
- [0x421 Automatic Transmission](#0x421-automatic-transmission)
- [0x4DF](#0x4df)
- [0x4CC ADAS Errors](#0x4cc-adas-errors)
- [0x54A HVAC #1](#0x54a-hvac-1)
- [0x54B HVAC #2](#0x54b-hvac-2)
- [0x54C HVAC #3](#0x54c-hvac-3)
- [0x54D HVAC #4](#0x54d-hvac-4)
- [0x551 Engine](#0x551-engine)
- [0x56C Chassis Control](#0x56c-chassis-control)
- [0x56E Telematics](#0x56e-telematics)
- [0x580 Engine](#0x580-engine)
- [0x5C5 Cluster Status #4](#0x5c5-cluster-status-4)
- [0x60D BCM](#0x60d-bcm)
- [0x625 BCM/IPDM](#0x625-bcm-or-ipdm)

---

## 0x002 Steering Angle

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x002</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Steering Angle</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">10 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Steering angle, signed little-endian, factor −0.1°</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" align="center">Absolute steering rate, factor ≈ 4°/s</td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td colspan="4" align="center">Rolling counter</td>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Steering angle, signed little-endian, factor −0.1° | Bytes 0–1 | 16 |
| Absolute steering rate, factor ≈ 4°/s | Byte 2 | 8 |
| Rolling counter | Byte 3 [3:0] | 4 |

---

## 0x160 Accelerator Pedal

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x160</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Pedal</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td align="center">Pedal fully pressed?</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>3</th>
    <td colspan="8" align="center">Accelerator position (% × 2)</td>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>5</th>
    <td colspan="8" align="center">Inverse accelerator position (0xFF − byte 3)</td>
  </tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Pedal fully pressed? | Byte 0 [6] | 1 |
| Accelerator position (% × 2) | Byte 3 | 8 |
| Inverse accelerator position (0xFF − byte 3) | Byte 5 | 8 |

---

## 0x180 Engine RPM

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x180</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Engine RPM</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">ENGINE_RPM</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>5</th>
    <td colspan="8" align="center">ACCELERATOR_POSITION (0.5% / bit)</td>
  </tr>
  <tr>
    <th>6</th>
    <td colspan="8" align="center">COUNTER</td>
  </tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| ENGINE_RPM | Bytes 0–1 | 16 |
| ACCELERATOR_POSITION (0.5% / bit) | Byte 5 | 8 |
| COUNTER | Byte 6 | 8 |

---

## 0x182

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x182</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>5</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td colspan="4" align="center">COUNTER (0:F)</td>
  </tr>
  <tr>
    <th>6</th>
    <td></td>
    <td align="center">Brake Pressed</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| COUNTER (0:F) | Byte 5 [3:0] | 4 |
| Brake Pressed | Byte 6 [6] | 1 |

---

## 0x1A5 AWD

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x1A5</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">AWD</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

> **Notes**
>
> - Adding then removing this message triggers AWD error unless B+ is cycled (car forgets it is AWD)

---

## 0x216 BCM

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x216</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">BCM</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">20 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="3" align="center">WIPER_STATE</td>
    <td></td>
    <td></td>
    <td colspan="3" align="center">OP_STATE</td>
  </tr>
  <tr>
    <th>1</th>
    <td></td>
    <td align="center">DRL on</td>
    <td align="center">Headlights</td>
    <td></td>
    <td align="center">Fog Lights</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| WIPER_STATE | Byte 0 [7:5] | 3 |
| OP_STATE | Byte 0 [2:0] | 3 |
| DRL on | Byte 1 [6] | 1 |
| Headlights | Byte 1 [5] | 1 |
| Fog Lights | Byte 1 [3] | 1 |

### 0x216 · OP_STATE

| Value | Meaning |
|-------|---------|
| 0b000 |  |
| 0b001 |  |
| 0b010 |  |
| 0b011 |  |
| 0b100 |  |
| 0b101 |  |
| 0b110 |  |
| 0b111 |  |

### 0x216 · WIPER_STATE

| Value | Meaning |
|-------|---------|
| 0b000 |  |
| 0b001 |  |
| 0b010 |  |
| 0b011 |  |
| 0b100 |  |
| 0b101 |  |
| 0b110 |  |
| 0b111 |  |

---

## 0x280 Cluster Status 1

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x280</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Meter Status?</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">20 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Driver Unbuckled</td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td colspan="8" align="center">Fuel Level Voltage Normalized Raw (10-bit)</td>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="2" align="center">↑ Fuel Level Voltage Normalized Raw</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>4</th>
    <td colspan="8" rowspan="2" align="center">Processed Vehicle Speed (km/h × 100)</td>
  </tr>
  <tr>
    <th>5</th>
  </tr>
  <tr>
    <th>6</th>
    <td colspan="8" align="center">Fuel Level Voltage ADC Raw Value (10-bit)</td>
  </tr>
  <tr>
    <th>7</th>
    <td colspan="2" align="center">↑ Fuel Level Voltage ADC Raw Value</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Driver Unbuckled | Byte 0 [1] | 1 |
| Fuel Level Voltage Normalized Raw (10-bit) | Byte 1 + Byte 2 [7:6] | 10 |
| Processed Vehicle Speed (km/h × 100) | Bytes 4–5 | 16 |
| Fuel Level Voltage ADC Raw Value (10-bit) | Byte 6 + Byte 7 [7:6] | 10 |

---

## 0x284 Front Wheel Speeds

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x284</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Front Wheel Speeds</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Front Right Wheel Speed (raw × 0.005 km/h)</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" rowspan="2" align="center">Front Left Wheel Speed</td>
  </tr>
  <tr>
    <th>3</th>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Front Right Wheel Speed (raw × 0.005 km/h) | Bytes 0–1 | 16 |
| Front Left Wheel Speed | Bytes 2–3 | 16 |


---

## 0x285 Rear Wheel Speeds

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x285</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Rear Wheel Speeds</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Rear Right Wheel Speed</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" rowspan="2" align="center">Rear Left Wheel Speed</td>
  </tr>
  <tr>
    <th>3</th>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Rear Right Wheel Speed | Bytes 0–1 | 16 |
| Rear Left Wheel Speed | Bytes 2–3 | 16 |

---

## 0x292 Forces

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x292</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">G Force & Brake</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">20 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" align="center">Longitudinal acceleration (raw × 0.001)</td>
  </tr>
  <tr>
    <th>1</th>
    <td colspan="4" align="center">↑ Longitudinal acceleration</td>
    <td colspan="4" align="center">Lateral acceleration ↓</td>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" align="center">Lateral acceleration (raw × 0.001)</td>
  </tr>
  <tr>
    <th>3</th>
    <td colspan="8" align="center">Yaw rate (raw × 0.1)</td>
  </tr>
  <tr>
    <th>4</th>
    <td colspan="4" align="center">↑ Yaw rate</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>6</th>
    <td colspan="8" align="center">Brake pressure (raw)</td>
  </tr>
  <tr>
    <th>7</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td colspan="2" align="center">Counter</td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Longitudinal acceleration (raw × 0.001) | Byte 0 + Byte 1 [7:4] | 12 |
| Lateral acceleration (raw × 0.001) | Byte 1 [3:0] + Byte 2 | 12 |
| Yaw rate (raw × 0.1) | Byte 3 + Byte 4 [7:4] | 12 |
| Brake pressure (raw) | Byte 6 | 8 |
| Counter | Byte 7 [1:0] | 2 |

---

## 0x2B1 ADAS

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x2B1</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">ADAS</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">6</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">20 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="2" align="center">CRUISE_CONTROL</td>
    <td align="center">CC Enabled</td>
    <td></td>
    <td colspan="2" align="center">BARS</td>
    <td align="center">Display MPH</td>
    <td align="center">Adaptive CC</td>
  </tr>
  <tr>
    <th>1</th>
    <td colspan="2" align="center">CAR_AHEAD</td>
    <td align="center">Lane Assist Warning Left</td>
    <td align="center">Lane Assist Warning Right</td>
    <td align="center">Blind Spot Left Blink</td>
    <td align="center">Blind Spot Right Blink</td>
    <td align="center">Emergency Brake Warning</td>
    <td align="center">Winding Road Icon</td>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="2" align="center">EMG_BRAKE</td>
    <td colspan="2" align="center">LANE_ASSIST</td>
    <td colspan="2" align="center">BLIND_SPOT</td>
    <td align="center">FEB Light Blink</td>
    <td align="center">FEB Light</td>
  </tr>
  <tr>
    <th>3</th>
    <td colspan="2" align="center">EMG_BRAKE_2</td>
    <td colspan="2" align="center">LANE_ASSIST_2</td>
    <td colspan="2" align="center">BLIND_SPOT_2</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>4</th>
    <td colspan="8" align="center">CC_SPEED</td>
  </tr>
  <tr>
    <th>5</th>
    <td colspan="3" align="center">TONES</td>
    <td align="center">Switch Page</td>
    <td align="center">Blink Speed</td>
    <td align="center">Prompt Page</td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| CRUISE_CONTROL | Byte 0 [7:6] | 2 |
| CC Enabled | Byte 0 [5] | 1 |
| BARS | Byte 0 [3:2] | 2 |
| Display MPH | Byte 0 [1] | 1 |
| Adaptive CC | Byte 0 [0] | 1 |
| CAR_AHEAD | Byte 1 [7:6] | 2 |
| Lane Assist Warning Left | Byte 1 [5] | 1 |
| Lane Assist Warning Right | Byte 1 [4] | 1 |
| Blind Spot Left Blink | Byte 1 [3] | 1 |
| Blind Spot Right Blink | Byte 1 [2] | 1 |
| Emergency Brake Warning | Byte 1 [1] | 1 |
| Winding Road Icon | Byte 1 [0] | 1 |
| EMG_BRAKE | Byte 2 [7:6] | 2 |
| LANE_ASSIST | Byte 2 [5:4] | 2 |
| BLIND_SPOT | Byte 2 [3:2] | 2 |
| FEB Light Blink | Byte 2 [1] | 1 |
| FEB Light | Byte 2 [0] | 1 |
| EMG_BRAKE_2 | Byte 3 [7:6] | 2 |
| LANE_ASSIST_2 | Byte 3 [5:4] | 2 |
| BLIND_SPOT_2 | Byte 3 [3:2] | 2 |
| CC_SPEED | Byte 4 | 8 |
| TONES | Byte 5 [7:5] | 3 |
| Switch Page | Byte 5 [4] | 1 |
| Blink Speed | Byte 5 [3] | 1 |
| Prompt Page | Byte 5 [2] | 1 |

### 0x2B1 · CRUISE_CONTROL

| Value | Meaning |
|-------|---------|
| 0b00 | None |
| 0b01 | CC Icon |
| 0b10 | CC Blink |
| 0b11 | Yellow CC (Warning) |

### 0x2B1 · BARS

| Value | Meaning |
|-------|---------|
| 0b00 | No Bars |
| 0b01 | 1 Bar |
| 0b10 | 2 Bars |
| 0b11 | 3 Bars |

### 0x2B1 · CAR_AHEAD

| Value | Meaning |
|-------|---------|
| 0b00 | None |
| 0b01 | White Car Ahead |
| 0b10 | Yellow Blinking Car |
| 0b11 | Crash Alert |

### 0x2B1 · EMG_BRAKE

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Enabled |
| 0b10 | White Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · LANE_ASSIST

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Enabled |
| 0b10 | White Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · BLIND_SPOT

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Enabled |
| 0b10 | White Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · EMG_BRAKE_2

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Green Enabled |
| 0b10 | Green Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · LANE_ASSIST_2

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Green Enabled |
| 0b10 | Green Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · BLIND_SPOT_2

| Value | Meaning |
|-------|---------|
| 0b00 | Disabled |
| 0b01 | Green Enabled |
| 0b10 | Green Blink (Warning Icon) |
| 0b11 | Yellow Solid (Warning Icon) |

### 0x2B1 · CC_SPEED

| Value | Meaning |
|-------|---------|
| 0x00 | Blank |
| 0x01–0xFE | Speed − 1 |
| 0xFF | `- - -` |

### 0x2B1 · TONES

| Value | Meaning |
|-------|---------|
| 0b000 | None |
| 0b001 | Constant beep |
| 0b010 | Quick beep |
| 0b011 | Triple fast beep |
| 0b100 | Fast single beep |
| 0b101 | Quad beep |
| 0b110 | Single beep |
| 0b111 | Double beep |

---

## 0x2DE Cluster Status #2

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x2DE</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Cluster Status?</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td></td>
    <td align="center">Paddle Shift Down Req</td>
    <td align="center">Paddle Shift Up Req</td>
    <td align="center">Stick Shift Down Req</td>
    <td align="center">Stick Shift Up Req</td>
    <td align="center">Auto Mode?</td>
    <td align="center">Manual Mode</td>
  </tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>3</th>
    <td colspan="8" align="center">Average Fuel Economy (km/L × 100)</td>
  </tr>
  <tr>
    <th>4</th>
    <td colspan="4" align="center">↑ Average Fuel Economy</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>6</th>
    <td colspan="8" rowspan="2" align="center">Distance to Empty (km × 0.1)</td>
  </tr>
  <tr>
    <th>7</th>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Paddle Shift Down Req | Byte 0 [5] | 1 |
| Paddle Shift Up Req | Byte 0 [4] | 1 |
| Stick Shift Down Req | Byte 0 [3] | 1 |
| Stick Shift Up Req | Byte 0 [2] | 1 |
| Auto Mode? | Byte 0 [1] | 1 |
| Manual Mode | Byte 0 [0] | 1 |
| Average Fuel Economy (km/L × 100) | Byte 3 + Byte 4 [7:4] | 12 |
| Distance to Empty (km × 0.1) | Bytes 6–7 | 16 |

---

## 0x351 Intelligent Key

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x351</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>1</th>
    <td align="center">UNLOCK</td>
    <td align="center">LOCK</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>5</th>
    <td align="center">Steering Lock Registration Complete</td>
    <td align="center">Key System Error</td>
    <td align="center">Double Beep</td>
    <td align="center">Fast Beep</td>
    <td colspan="4" align="center">KEY_PROMPTS</td>
  </tr>
  <tr>
    <th>6</th>
    <td align="center">Beep</td>
    <td></td>
    <td></td>
    <td align="center">Auto High Beam</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| UNLOCK | Byte 1 [7] | 1 |
| LOCK | Byte 1 [6] | 1 |
| Steering Lock Registration Complete | Byte 5 [7] | 1 |
| Key System Error | Byte 5 [6] | 1 |
| Double Beep | Byte 5 [5] | 1 |
| Fast Beep | Byte 5 [4] | 1 |
| KEY_PROMPTS | Byte 5 [3:0] | 4 |
| Beep | Byte 6 [7] | 1 |
| Auto High Beam | Byte 6 [4] | 1 |

### 0x351 · KEY_PROMPTS

| Value | Meaning |
|-------|---------|
| 0x0 | None |
| 0x1 | Brake + Push Start |
| 0x2 | No visible response |
| 0x3 | Key ID Incorrect |
| 0x4 | Turn Steering Wheel + Push Button |
| 0x5 | Shift to Park |
| 0x6 | Power turned off to save battery |
| 0x7 | Key Battery Low |
| 0x8 | No key detected |
| 0x9 | Power will turn off to save battery |
| 0xA | Push Ignition to OFF |
| 0xB | Clutch + Push Start |
| 0xC | No visible response |
| 0xD | No visible response |
| 0xE | Touch Key to Start Button |
| 0xF | Push Brake and Start Button to Drive |

---

## 0x354 Chassis Control

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x354</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">50 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Vehicle Speed (km/h × 100)</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>4</th>
    <td colspan="3" align="center">TCS_STATE</td>
    <td align="center">ABS</td>
    <td align="center">ABS latch</td>
    <td align="center">BRAKE</td>
    <td align="center">BRAKE latch</td>
    <td></td>
  </tr>
  <tr>
    <th>5</th>
    <td></td>
    <td></td>
    <td></td>
    <td colspan="2" align="center">COUNTER</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>6</th>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">BRAKE Pressed</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Vehicle Speed (km/h × 100) | Bytes 0–1 | 16 |
| TCS_STATE | Byte 4 [7:5] | 3 |
| ABS | Byte 4 [4] | 1 |
| ABS latch | Byte 4 [3] | 1 |
| BRAKE | Byte 4 [2] | 1 |
| BRAKE latch | Byte 4 [1] | 1 |
| COUNTER | Byte 5 [4:3] | 4 |
| BRAKE Pressed | Byte 6 [4] | 1 |

### 0x354 · TCS_STATE

| Value | Meaning |
|-------|---------|
| 0b000 | None |
| 0b001 | TCS Off Solid + TCS Solid |
| 0b010 | TCS Off Solid |
| 0b011 | TCS Blinking |
| 0b100 | TCS Blinking |
| 0b101 | TCS Solid |
| 0b110 | TCS Off + TCS Blinking |
| 0b111 | TCS Off + TCS Blinking |

---

## 0x355 Cluster Status #3

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x355</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Cluster Status #3</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Processed Vehicle Speed (km/h × 100)</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" rowspan="2" align="center">Displayed Vehicle Speed (km/h × 100)</td>
  </tr>
  <tr>
    <th>3</th>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Processed Vehicle Speed (km/h × 100) | Bytes 0–1 | 16 |
| Displayed Vehicle Speed (km/h × 100) | Bytes 2–3 | 16 |

---

## 0x358

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x358</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">?</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>1</th>
    <td align="center">Dim Backlight</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>2</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Right Blinker Request</td>
    <td align="center">Left Blinker Request</td>
    <td align="center">Trunk Open</td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td align="center">Low Oil Pressure</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>4</th>
    <td align="center">TPMS Light</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Dim Backlight | Byte 1 [7] | 1 |
| Right Blinker Request | Byte 2 [2] | 1 |
| Left Blinker Request | Byte 2 [1] | 1 |
| Trunk Open | Byte 2 [0] | 1 |
| Low Oil Pressure | Byte 3 [5] | 1 |
| TPMS Light | Byte 4 [7] | 1 |

---

## 0x35D

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x35D</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>4</th>
    <td></td>
    <td align="center">?</td>
    <td></td>
    <td align="center">Brake Pressed</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| ? | Byte 4 [6] | 1 |
| Brake Pressed | Byte 4 [4] | 1 |

---

## 0x385 TPMS Info

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x385</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">TPMS Info</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">0 = Tire pressures appear while driving</td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td align="center">FR Flat Tire</td>
    <td align="center">FL Flat Tire</td>
    <td align="center">RR Flat Tire</td>
    <td align="center">RL Flat Tire</td>
    <td align="center">FR Tire Warning</td>
    <td align="center">FL Tire Warning</td>
    <td align="center">RR Tire Warning</td>
    <td align="center">RL Tire Warning</td>
  </tr>
  <tr>
    <th>2</th>
    <td colspan="8" align="center">Front Right Tire PSI × 4</td>
  </tr>
  <tr>
    <th>3</th>
    <td colspan="8" align="center">Front Left Tire PSI × 4</td>
  </tr>
  <tr>
    <th>4</th>
    <td colspan="8" align="center">Rear Right Tire PSI × 4</td>
  </tr>
  <tr>
    <th>5</th>
    <td colspan="8" align="center">Rear Left Tire PSI × 4</td>
  </tr>
  <tr>
    <th>6</th>
    <td align="center">FR Valid</td>
    <td align="center">FL Valid</td>
    <td align="center">RR Valid</td>
    <td align="center">RL Valid</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>7</th>
    <td colspan="4" align="center">Target kPa Front (190 + N × 10)</td>
    <td colspan="4" align="center">Target kPa Rear (190 + N × 10)</td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| 0 = Tire pressures appear while driving | Byte 0 [1] | 1 |
| FR Flat Tire | Byte 1 [7] | 1 |
| FL Flat Tire | Byte 1 [6] | 1 |
| RR Flat Tire | Byte 1 [5] | 1 |
| RL Flat Tire | Byte 1 [4] | 1 |
| FR Tire Warning | Byte 1 [3] | 1 |
| FL Tire Warning | Byte 1 [2] | 1 |
| RR Tire Warning | Byte 1 [1] | 1 |
| RL Tire Warning | Byte 1 [0] | 1 |
| Front Right Tire PSI × 4 | Byte 2 | 8 |
| Front Left Tire PSI × 4 | Byte 3 | 8 |
| Rear Right Tire PSI × 4 | Byte 4 | 8 |
| Rear Left Tire PSI × 4 | Byte 5 | 8 |
| FR Valid | Byte 6 [7] | 1 |
| FL Valid | Byte 6 [6] | 1 |
| RR Valid | Byte 6 [5] | 1 |
| RL Valid | Byte 6 [4] | 1 |
| Target kPa Front (190 + N × 10) | Byte 7 [7:4] | 4 |
| Target kPa Rear (190 + N × 10) | Byte 7 [3:0] | 4 |

---

## 0x3AF

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x3AF</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">1</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td align="center">Hazard Button Pressed</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Hazard Button Pressed | Byte 0 [7] | 1 |

---

## 0x421 Automatic Transmission

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x421</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Automatic Transmission</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">3</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">60 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="5" align="center">GEARS</td>
    <td align="center">AT Error</td>
    <td></td>
    <td align="center">AT Error</td>
  </tr>
  <tr>
    <th>1</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Flash Gear</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>2</th>
    <td></td>
    <td align="center">Beep</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| GEARS | Byte 0 [7:3] | 5 |
| AT Error | Byte 0 [2] | 1 |
| AT Error | Byte 0 [0] | 1 |
| Flash Gear | Byte 1 [2] | 1 |
| Beep | Byte 2 [6] | 1 |

### 0x421 · GEARS

| Value | Meaning |
|-------|---------|
| 0b00000 | No Gear |
| 0b00001 | Park |
| 0b00010 | Reverse |
| 0b00011 | Neutral |
| 0b00100 | Drive |
| 0b00101 | Drive (S) |
| 0b00110 | Low |
| 0b00111 |  |
| 0b01000 | 1 |
| 0b01001 | 2 |
| 0b01010 | 3 |
| 0b01011 | 4 |
| 0b01100 | 5 |
| 0b01101 | 6 |
| 0b01110 |  |
| 0b01111 |  |
| 0b10000 | 1 (M) |
| 0b10001 | 2 (M) |
| 0b10010 | 3 (M) |
| 0b10011 | 4 (M) |
| 0b10100 | 5 (M) |
| 0b10101 | 6 (M) |
| 0b10110 | 7 (M) |
| 0b10111 | 8 (M) |

---

## 0x4DF

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x4DF</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

---

## 0x4CC ADAS Errors

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x4CC</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">ADAS Errors</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">5</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">500 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Unavailable High Accelerator Temperature - DCA</td>
    <td align="center">Unavailable High Accelerator Temperature - BCI</td>
    <td align="center">Unavailable High Accelerator Temperature - FEB</td>
  </tr>
  <tr>
    <th>1</th>
    <td align="center">Unavailable Clean Rear Camera - LDW</td>
    <td align="center">Unavailable Clean Rear Camera - BSW</td>
    <td align="center">Malfunction - PFCW</td>
    <td align="center">Malfunction - LDW</td>
    <td align="center">Malfunction - BSW</td>
    <td align="center">Unavailable High Cabin Temperature - LDW</td>
    <td align="center">Unavailable High Cabin Temperature - LDP</td>
    <td align="center">Unavailable High Cabin Temperature - BSI</td>
  </tr>
  <tr>
    <th>2</th>
    <td align="center">Unavailable Front Radar Obstruction - ICC</td>
    <td align="center">Unavailable Front Radar Obstruction - PFCW</td>
    <td align="center">Unavailable Front Radar Obstruction - DCA</td>
    <td align="center">Unavailable Front Radar Obstruction - FEB</td>
    <td align="center">Unavailable Select Driving Aids in Settings</td>
    <td align="center">Unavailable Select Driving Aids in Settings</td>
    <td align="center">Currently Unavailable - ICC</td>
    <td align="center">Currently Unavailable - ICC</td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Unavailable Side Radar Obstruction - BSW</td>
    <td align="center">Unavailable Side Radar Obstruction - BSI</td>
    <td align="center">Unavailable Side Radar Obstruction - BCI</td>
    <td></td>
  </tr>
  <tr>
    <th>4</th>
    <td align="center">Malfunction - DCA</td>
    <td align="center">Malfunction - LDP</td>
    <td align="center">Malfunction - BSI</td>
    <td align="center">Malfunction - FEB</td>
    <td align="center">Malfunction - BCI</td>
    <td align="center">System OFF - BCI</td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Unavailable High Accelerator Temperature - DCA | Byte 0 [2] | 1 |
| Unavailable High Accelerator Temperature - BCI | Byte 0 [1] | 1 |
| Unavailable High Accelerator Temperature - FEB | Byte 0 [0] | 1 |
| Unavailable Clean Rear Camera - LDW | Byte 1 [7] | 1 |
| Unavailable Clean Rear Camera - BSW | Byte 1 [6] | 1 |
| Malfunction - PFCW | Byte 1 [5] | 1 |
| Malfunction - LDW | Byte 1 [4] | 1 |
| Malfunction - BSW | Byte 1 [3] | 1 |
| Unavailable High Cabin Temperature - LDW | Byte 1 [2] | 1 |
| Unavailable High Cabin Temperature - LDP | Byte 1 [1] | 1 |
| Unavailable High Cabin Temperature - BSI | Byte 1 [0] | 1 |
| Unavailable Front Radar Obstruction - ICC | Byte 2 [7] | 1 |
| Unavailable Front Radar Obstruction - PFCW | Byte 2 [6] | 1 |
| Unavailable Front Radar Obstruction - DCA | Byte 2 [5] | 1 |
| Unavailable Front Radar Obstruction - FEB | Byte 2 [4] | 1 |
| Unavailable Select Driving Aids in Settings | Byte 2 [3] | 1 |
| Unavailable Select Driving Aids in Settings | Byte 2 [2] | 1 |
| Currently Unavailable - ICC | Byte 2 [1] | 1 |
| Currently Unavailable - ICC | Byte 2 [0] | 1 |
| Unavailable Side Radar Obstruction - BSW | Byte 3 [3] | 1 |
| Unavailable Side Radar Obstruction - BSI | Byte 3 [2] | 1 |
| Unavailable Side Radar Obstruction - BCI | Byte 3 [1] | 1 |
| Malfunction - DCA | Byte 4 [7] | 1 |
| Malfunction - LDP | Byte 4 [6] | 1 |
| Malfunction - BSI | Byte 4 [5] | 1 |
| Malfunction - FEB | Byte 4 [4] | 1 |
| Malfunction - BCI | Byte 4 [3] | 1 |
| System OFF - BCI | Byte 4 [2] | 1 |

| Acronym | Meaning | Description |
|--------|----------|-------------|
| ALC | Active Lane Control | Steers the wheel to maintain lane position
| LDW | Lane Departure Warning | Signals when deviation from lane is detected
| LDP | Lane Departure Prevention | Takes control of steering and centers vehicle when deviation from lane is detected
| BSW | Blind Spot Warning | Signals when vehicle is in blind spot
| BSI | Blind Spot Intervention | Takes control of steering and centers vehicle when switching lanes with vehicle in blind spot
| BCI | Back-up Collision Intervention | Brakes the vehicle if cross-traffic is detected when backing up 
| DCA | Distance Control Assist | Maintains distances from vehicle ahead
| FEB | Forward Emergency Braking | Applies brakes if car is moving towards a stopped vehicle 
| PFCW | Predictive Forward Collision Warning | Signals if traffic stops suddenly a few cars ahead
| ICC | Intelligent Cruise Control | Maintains cruise control speed and decelerates if necessary to maintain a safe distance from vehicle ahead

---

## 0x54A HVAC #1

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x54A</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">HVAC #1</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>4</th>
    <td colspan="8" align="center">Driver HVAC Temp</td>
  </tr>
  <tr>
    <th>5</th>
    <td colspan="8" align="center">Passenger HVAC Temp</td>
  </tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Driver HVAC Temp | Byte 4 | 8 |
| Passenger HVAC Temp | Byte 5 | 8 |

---

## 0x54B HVAC #2

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x54B</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">HVAC #2</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>2</th>
    <td></td>
    <td></td>
    <td colspan="3" align="center">AIR_DIRECTION</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td align="center">Dual</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Recirculation</td>
    <td></td>
  </tr>
  <tr>
    <th>4</th>
    <td></td>
    <td></td>
    <td colspan="4" align="center">BLOWER_SPEED</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>5</th>
    <td></td>
    <td align="center">Steering Heater</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| AIR_DIRECTION | Byte 2 [5:3] | 3 |
| Dual | Byte 3 [6] | 1 |
| Recirculation | Byte 3 [1] | 1 |
| BLOWER_SPEED | Byte 4 [5:2]| 4 |
| Steering Heater | Byte 5 [6] | 1 |

### 0x54B · AIR_DIRECTION

| Value | Meaning |
|-------|---------|
| 0b001 | Upper |
| 0b010 | Upper + Lower |
| 0b011 | Lower |
| 0b100 | Defogger + Lower |
| 0b101 | Defogger |

### 0x54B · BLOWER_SPEED

| Value | Meaning |
|-------|---------|
| 0x0C | Level 1 |
| 0x14 | Level 2 |
| 0x1C | Level 3 |
| 0x24 | Level 4 |
| 0x2C | Level 5 |
| 0x34 | Level 6 |
| 0x3C | Level 7 |

---

## 0x54C HVAC #3

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x54C</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">HVAC #3</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>1</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

---

## 0x54D HVAC #4

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x54D</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">HVAC #4</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7"></td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr><th>0</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>1</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Driver Seat Heat AUTO</td>
    <td align="center">Passenger Seat Heat AUTO</td>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>5</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Driver Seat Heat OFF</td>
    <td align="center">Driver Seat Heat LOW</td>
    <td align="center">Driver Seat Heat MID</td>
  </tr>
  <tr>
    <th>6</th>
    <td align="center">Driver Seat Heat HIGH</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Passenger Seat Heat OFF</td>
    <td align="center">Passenger Seat Heat LOW</td>
    <td align="center">Passenger Seat Heat MID</td>
  </tr>
  <tr>
    <th>7</th>
    <td align="center">Passenger Seat Heat HIGH</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Driver Seat Heat AUTO | Byte 1 [1] | 1 |
| Passenger Seat Heat AUTO | Byte 1 [0] | 1 |
| Driver Seat Heat OFF | Byte 5 [2] | 1 |
| Driver Seat Heat LOW | Byte 5 [1] | 1 |
| Driver Seat Heat MID | Byte 5 [0] | 1 |
| Driver Seat Heat HIGH | Byte 6 [7] | 1 |
| Passenger Seat Heat OFF | Byte 6 [2] | 1 |
| Passenger Seat Heat LOW | Byte 6 [1] | 1 |
| Passenger Seat Heat MID | Byte 6 [0] | 1 |
| Passenger Seat Heat HIGH | Byte 7 [7] | 1 |

---

## 0x551 Engine

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x551</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Engine</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" align="center">Coolant Temperature</td>
  </tr>
  <tr>
    <th>1</th>
    <td colspan="8" rowspan="2" align="center">CUMULATIVE_FUEL_USAGE_COUNTER</td>
    <tr><th>2</th></tr>
  </tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Coolant Temperature | Byte 0 | 8 |
| CUMULATIVE_FUEL_USAGE_COUNTER | Byte 1 | 8 |

---

## 0x56C Chassis Control

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x56C</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Chassis Control</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">5</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td align="center">Trace control blue squares</td>
    <td align="center">Trace control 2 squares wheels</td>
    <td align="center">Trace control blue lines on wheels</td>
    <td></td>
    <td></td>
    <td align="center"></td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td></td>
    <td align="center">Trace Control Saddle Curve</td>
    <td colspan="3" align="center">DRIVE_MODES</td>
    <td align="center">Quad Beep</td>
    <td align="center">Continuous Beep</td>
    <td></td>
  </tr>
  <tr>
    <th>2</th>
    <td></td>
    <td align="center">Front Left Wheel</td>
    <td></td>
    <td align="center">Front Right Wheel</td>
    <td></td>
    <td align="center">Rear Left Wheel</td>
    <td></td>
    <td align="center">Rear Right Wheel</td>
  </tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Trace control blue squares | Byte 0 [6] | 1 |
| Trace control 2 squares wheels | Byte 0 [5] | 1 |
| Trace control blue lines on wheels | Byte 0 [4] | 1 |
| Trace control saddle curve | Byte 1 [6] | 1 |
| DRIVE_MODES | Byte 1 [5:3] | 3 |
| Quadruple Beep | Byte 1 [2] | 1 |
| Continuous Beep | Byte 1 [1] | 1 |
| Front Left Wheel | Byte 2 [6] | 1 |
| Front Right Wheel | Byte 2 [4] | 1 |
| Rear Left Wheel | Byte 2 [2] | 1 |
| Rear Right Wheel | Byte 2 [0] | 1 |

### 0x56C · DRIVE_MODES

| Value | Meaning |
|-------|---------|
| 0b000 | Standard |
| 0b001 | Sport |
| 0b010 | Eco |
| 0b011 | Snow |
| 0b100 | Personal |
| 0b101 | Sport Plus |

> **Notes**
>
> - need to verify DLC

---

## 0x56E Telematics

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x56E</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Telematics</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">4</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td align="center">?</td>
    <td align="center">?</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Unlock Command</td>
  </tr>
  <tr>
    <th>1</th>
    <td align="center">Lock Command</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>3</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| ? | Byte 0 [7] | 1 |
| ? | Byte 0 [6] | 1 |
| Unlock Command | Byte 0 [0] | 1 |
| Lock Command | Byte 1 [7] | 1 |

> **Notes**
>
> - Baseline: `86 00 00 00`.

---

## 0x580 Engine

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x580</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7"></td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td colspan="8" rowspan="2" align="center">Engine Torque (raw / 16 Nm)</td>
  </tr>
  <tr>
    <th>1</th>
  </tr>
  <tr>
    <th>2</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Low Oil Pressure</td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">Loose Fuel Cap</td>
  </tr>
  <tr>
    <th>4</th>
    <td colspan="8" align="center">Engine Oil Temp (raw °C + 40)</td>
  </tr>
  <tr>
    <th>5</th>
    <td></td>
    <td align="center">ECO mode light</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Engine Torque (raw / 16 Nm) | Bytes 0–1 | 16 |
| Low Oil Pressure | Byte 2 [0] | 1 |
| Loose Fuel Cap | Byte 3 [0] | 1 |
| Engine Oil Temp (raw °C + 40) | Byte 4 | 8 |
| ECO mode light | Byte 5 [6] | 1 |

---

## 0x5C5 Cluster Status #4

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x5C5</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">Meter Status #4</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td align="center">Meter OFF</td>
    <td align="center">Meter ON</td>
    <td align="center">Gas Light Status</td>
    <td></td>
    <td align="center">Low Brake Fluid</td>
    <td align="center">Parking Brake</td>
    <td align="center">Running?</td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td colspan="8" rowspan="3" align="center">Odometer Mileage</td>
  </tr>
  <tr>
    <th>2</th>
  </tr>
  <tr>
    <th>3</th>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Meter OFF | Byte 0 [7] | 1 |
| Meter ON | Byte 0 [6] | 1 |
| Gas Light Status | Byte 0 [5] | 1 |
| Low Brake Fluid | Byte 0 [3] | 1 |
| Parking Brake | Byte 0 [2] | 1 |
| Running? | Byte 0 [1] | 1 |
| Odometer Mileage | Bytes 1–3 | 24 |

---

## 0x60D BCM

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x60D</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">BCM</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">8</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td align="center">Trunk</td>
    <td align="center">Rear Passenger Door</td>
    <td align="center">Rear Driver Door</td>
    <td align="center">Passenger Door</td>
    <td align="center">Driver Door</td>
    <td align="center">Parking Lights</td>
    <td align="center">Headlights</td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td></td>
    <td align="center">Right Turn Signal</td>
    <td align="center">Left Turn Signal</td>
    <td></td>
    <td align="center">High Beam</td>
    <td align="center">IG/RUN</td>
    <td align="center">ACC</td>
    <td align="center">Fog Lights</td>
  </tr>
  <tr>
    <th>2</th>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">All doors locked</td>
    <td></td>
    <td align="center">Rear Fog Light</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td colspan="5" align="center">CHIMES</td>
    <td></td>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>6</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>7</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| Trunk | Byte 0 [7] | 1 |
| Rear Passenger Door | Byte 0 [6] | 1 |
| Rear Driver Door | Byte 0 [5] | 1 |
| Passenger Door | Byte 0 [4] | 1 |
| Driver Door | Byte 0 [3] | 1 |
| Parking Lights | Byte 0 [2] | 1 |
| Headlights | Byte 0 [1] | 1 |
| Right Turn Signal | Byte 1 [6] | 1 |
| Left Turn Signal | Byte 1 [5] | 1 |
| High Beam | Byte 1 [3] | 1 |
| IG/RUN | Byte 1 [2] | 1 |
| ACC | Byte 1 [1] | 1 |
| Fog Lights | Byte 1 [0] | 1 |
| All doors locked | Byte 2 [4] | 1 |
| Rear Fog Light | Byte 2 [2] | 1 |
| CHIMES | Byte 3 [5:1] | 5 |


### 0x60D · CHIMES

| Value | Meaning |
|-------|---------|
| 0b00000 | None |
| 0b00001 | Headlight Reminder |
| 0b10000 | Door Open Chime |
| 0b10010 | Single Short High Pitch Beep |
| 0b10011 | Periodic Quad Beep |
| 0b10101 | Startup Beep |
| 0b11000 | Long Constant Beep |
| 0b11001 | Long Constant Beep |

---

## 0x625 BCM or IPDM

<table>
  <tr>
    <th colspan="2" align="left">CAN ID</th>
    <td colspan="7">0x625</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Description</th>
    <td colspan="7">BCM or IPDM</td>
  </tr>
  <tr>
    <th colspan="2" align="left">DLC</th>
    <td colspan="7">6</td>
  </tr>
  <tr>
    <th colspan="2" align="left">Interval</th>
    <td colspan="7">100 ms</td>
  </tr>

  <tr><td colspan="9"></td></tr>   <!-- spacer -->

  <tr>
    <th>Byte</th>
    <th>Bit 7</th><th>Bit 6</th><th>Bit 5</th><th>Bit 4</th>
    <th>Bit 3</th><th>Bit 2</th><th>Bit 1</th><th>Bit 0</th>
  </tr>
  <tr>
    <th>0</th>
    <td></td>
    <td></td>
    <td colspan="2" align="center">START_STATE</td>
    <td></td>
    <td align="center">Wiper on low</td>
    <td align="center">Wipers not at top</td>
    <td></td>
  </tr>
  <tr>
    <th>1</th>
    <td align="center">AC Compressor OR Front Fan?</td>
    <td align="center">Parking Lights</td>
    <td align="center">Headlight</td>
    <td align="center">High Beam</td>
    <td align="center">Fog Lights</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>2</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr>
    <th>3</th>
    <td></td>
    <td></td>
    <td></td>
    <td align="center">IG/RUN</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr><th>4</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
  <tr><th>5</th><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
</table>

| Signal | Position | Length |
|--------|----------|--------|
| START_STATE | Byte 0 [5:4] | 2 |
| Wiper on low | Byte 0 [2] | 1 |
| Wipers not at top | Byte 0 [1] | 1 |
| AC Compressor OR Front Fan | Byte 1 [7] | 1 |
| Parking Lights | Byte 1 [6] | 1 |
| Headlight | Byte 1 [5] | 1 |
| High Beam | Byte 1 [4] | 1 |
| Fog Lights | Byte 1 [3] | 1 |
| IG/RUN | Byte 3 [4] | 1 |

### 0x625 · START_STATE

| Value | Meaning |
|-------|---------|
| 0b00 | Normal |
| 0b01 | ? |
| 0b10 | ? |
| 0b11 | Cranking |
