---
name: ethercat-diagnostics
description: "Use to diagnose an EtherCAT network: working counter (WKC) errors, lost frames, slaves stuck in INIT, PREOP or SAFEOP, AL status codes, CRC and link-lost error counters, locating a bad cable or connector from the counters, topology mismatches, and distributed clock (DC) sync problems. Not for drive state machines (use cia402-motion), NC axis configuration (use twincat-motion), PLC code (use twincat-st), or reading and checking ESI/ENI device description files (use ethercat-esi). DRAFT: under development."
---

# ethercat-diagnostics

> **Draft.** This skill is under development and not ready to install.

## Goal

Find where and why an EtherCAT network is failing — which slave, which port, which
cause — from the diagnostics the master exposes.

## Rules

- Answer from the facts below or from data the user gives you. **Never guess a register
  address, value or code.** If it is not here and not in the user's data, say so and point
  to the device documentation.
- Name a slave and a port, not "check the network". The answer a technician needs is
  "reseat the cable between slave 2 and slave 3".
- Ask for the readings you need (states, AL status code, error counters) instead of
  listing everything that could be wrong.

## Workflow

1. **States.** Are all slaves in OP? The first slave below OP, or with the error bit set,
   is the place to start.
2. **Why it refused.** For a slave below OP, read the AL status code and look it up below.
3. **Counters.** If states are fine but the WKC is wrong or frames are lost, read the error
   and lost-link counters of every slave and locate the first one with real (not forwarded)
   errors.
4. **Identity.** If a slave stays in INIT or PREOP, compare its identity with the
   configured device.
5. **Clocks.** If AL codes point at sync or DC, check the system time difference.

## What EtherCAT is not

Generic fieldbus advice gets these wrong; correct them when they come up.

- EtherCAT runs on 100 Mbit/s Ethernet. There is **no baud rate, word length or parity**
  setting.
- Slaves have **no IP address** (exception: Ethernet over EtherCAT, EoE). They are
  addressed by their position in the ring or by a configured station address
  (ESC register 0x0010).
- Clock sync uses **Distributed Clocks (DC)**, not PTP.
- EtherNet/IP is a different fieldbus.

## Topology basics

- A frame leaves the master, passes through every slave and returns to the master.
- Port 0 is IN (towards the master), port 1 is OUT (towards the next slave), ports 2 and 3
  are branches.

## Slave state — ESC 0x0130 (AL Status)

Low four bits are the state; bit 4 (0x10) is the error indicator.

| Value | State |
|---|---|
| 0x01 | INIT |
| 0x02 | PREOP |
| 0x03 | BOOT |
| 0x04 | SAFEOP |
| 0x08 | OP |

Examples: 0x14 is SAFEOP with error, 0x12 is PREOP with error. In normal operation every
slave reads 0x08. ESC 0x0120 (AL Control) holds the state the master requested; if it
differs from AL Status, the slave refused the transition.

## Why a slave refused — ESC 0x0134 (AL Status Code)

0x0000 means no error. The codes that come up most in the field:

| Code | Meaning | What to check |
|---|---|---|
| 0x0011 | Invalid requested state change | Master requested a jump the slave does not allow; restart from INIT |
| 0x0016 | Invalid mailbox configuration | Mailbox sync manager settings vs. the device |
| 0x0017 | Invalid sync manager configuration | Sync manager settings vs. the device and its revision |
| 0x001A | Synchronization error | Sync mode / DC settings, frame loss |
| 0x001B | Sync manager watchdog | Process data did not arrive in time: PLC stopped, cycle too slow, frames lost |
| 0x001D | Invalid output configuration | Output process data settings |
| 0x001E | Invalid input configuration | Input process data settings |
| 0x0024 | Invalid input mapping | Input PDO mapping does not match the device |
| 0x0025 | Invalid output mapping | Output PDO mapping does not match the device |
| 0x002C | Fatal sync error | DC sync lost; check DC settings and frame loss |
| 0x0030 | Invalid DC SYNC configuration | DC settings vs. what the device supports |
| 0x0032 | PLL error | DC not locked to the reference clock |
| 0x0035 | Invalid DC sync cycle time | Cycle time not supported by the device |

For any other code, look it up in the device documentation; do not guess.

## Link status — ESC 0x0110 (DL Status)

Bits 4–7 are physical link on ports 0–3 (1 = link). Expect link only on ports that have a
cable plugged in. A missing link on a cabled port is the fault location.

## Error counters

All are 8-bit and stop at 0xFF. Writing to 0x0300 clears 0x0300–0x030B.

| Register | Counter |
|---|---|
| 0x0300 / 0x0301 | Port 0 invalid frame / RX error |
| 0x0302 / 0x0303 | Port 1 invalid frame / RX error |
| 0x0304 / 0x0305 | Port 2 invalid frame / RX error |
| 0x0306 / 0x0307 | Port 3 invalid frame / RX error |
| 0x0308–0x030B | Forwarded RX error, ports 0–3 |
| 0x030C | Processing unit error |
| 0x0310–0x0313 | Lost link, ports 0–3 |

Healthy: zero, or not increasing. A few counts after power-up or after plugging a cable are
normal — clear the counters, then watch whether they climb.

## Locating a bad cable or connector

- A slave that receives a frame already marked bad counts a **forwarded** error, not an RX
  error. So find the **first slave from the master with RX or invalid-frame errors that are
  not forwarded** — everything after it only reports the damage.
- RX errors on **port 0 of slave N** → the cable or connectors between slave N−1 and N.
- RX errors on **port 1 of slave N** → the cable or connectors between slave N and N+1
  (seen on the return path).
- A climbing **lost-link** counter → loose connector, damaged cable, or the neighbour losing
  power. If it rises when a motor or conveyor starts, suspect cable movement or EMI.
- Fix in this order: reseat the connectors, replace the cable, check shielding and
  grounding, then replace the slave.

## Working counter (WKC)

Every slave that processes a datagram increments the working counter; the master compares
it with the value it expects. In TwinCAT the input `WcState` is 0 when the data is valid
and 1 when it is not.

A wrong WKC means one of: a slave is not in OP, a slave is missing or unpowered, or frames
are being lost. If all slaves are in OP and lost frames are climbing, use the counters to
find the bad section; if no slave counter climbs, suspect the master side (network card,
cable to the first slave).

## Identity — CoE 0x1018

| Subindex | Field |
|---|---|
| 0x1018:01 | Vendor ID |
| 0x1018:02 | Product code |
| 0x1018:03 | Revision |
| 0x1018:04 | Serial number |

Vendor, product code and revision must match the configured device. A mismatch keeps the
slave in INIT or PREOP, or the master reports a topology or identity error. Object 0x10F3,
where supported, holds the device's diagnosis history.

## Distributed clocks

- ESC 0x0910: local system time (64-bit, ns).
- ESC 0x092C: difference to the reference clock (bit 31 is the sign). It should be small
  and stable — typically well under 1 µs (a rule of thumb, not a specified limit). Large or
  growing means DC is not locked.

## Where to look in TwinCAT

- Slave → **Online** tab: current and requested state, AL status code, CRC counters.
- Slave → **Advanced Settings → ESC Access**: read raw ESC registers.
- Master (EtherCAT device) → **Online** tab: lost frames, frame counters, topology.
