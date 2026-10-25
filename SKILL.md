---
name: ethercat-diagnostics
description: "Use to diagnose an EtherCAT network: working counter (WKC) errors, lost frames, slaves stuck in INIT, PREOP or SAFEOP, AL status codes, CRC and link-lost error counters, locating a bad cable or connector from the counters, topology mismatches, distributed clock (DC) sync problems, and the ESI/ENI device description files behind them: reading an ESI, which PDOs and objects a device offers, vendor ID, product code and revision mismatches, and configured-versus-found differences. Not for drive state machines (use cia402-motion), NC axis configuration (use twincat-motion), or PLC code (use twincat-st). DRAFT: under development."
---

# ethercat-diagnostics

> **Draft.** This skill is under development and not ready to install.

## Goal

Find where and why an EtherCAT network is failing — which slave, which port, which
cause — from the diagnostics the master exposes.
