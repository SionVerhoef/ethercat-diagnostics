---
name: ethercat-diagnostics
description: "Use to diagnose an EtherCAT network: working counter (WKC) errors, lost frames, slaves stuck in INIT, PREOP or SAFEOP, AL status codes, CRC and link-lost error counters, locating a bad cable or connector from the counters, topology mismatches, and distributed clock (DC) sync problems. Not for drive state machines (use cia402-motion), NC axis configuration (use twincat-motion), PLC code (use twincat-st), or reading and checking ESI/ENI device description files (use ethercat-esi). DRAFT: under development."
---

# ethercat-diagnostics

> **Draft.** This skill is under development and not ready to install.

## Goal

Find where and why an EtherCAT network is failing — which slave, which port, which
cause — from the diagnostics the master exposes.
