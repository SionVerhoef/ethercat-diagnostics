# Eval questions

Hand-scored questions for checking an assistant against this skill — Claude with the skill
loaded, or a small local model with `SKILL.md` pasted in as context. Run each question
twice (small models are inconsistent), and once without the skill to see how much it adds.

The **traps** decide the outcome: a model that invents an answer to two or more of them is
not safe in front of a technician, whatever it scores elsewhere.

## Lookups

| # | Question | Pass if the answer says |
|---|---|---|
| 1 | Slave 4 shows AL status 0x14. What does that mean? | SAFEOP with the error bit set; read 0x0134 for the reason |
| 2 | AL status code is 0x001B. What do I check? | Sync manager watchdog: process data late — PLC running?, cycle time, lost frames |
| 3 | A drive reports AL status code 0x0035. | DC sync cycle time not supported; change cycle time or DC settings |
| 4 | How do I clear the CRC counters? | Write to 0x0300 (clears 0x0300–0x030B) |
| 5 | How do I check the right device is installed? | CoE 0x1018: vendor, product code, revision against the configuration |

## Reasoning

**6.** Given these counters, where is the fault?

```
Slave1: P0 RX=0  P1 RX=0  Fwd P0=0
Slave2: P0 RX=0  P1 RX=0  Fwd P0=0
Slave3: P0 RX=37 P1 RX=0  Fwd P0=0
Slave4: P0 RX=0  P1 RX=0  Fwd P0=35
Slave5: P0 RX=0  P1 RX=0  Fwd P0=34
```

Pass: cable or connectors between slave 2 and slave 3; slaves 4 and 5 only forward the
error. Fail: blames slave 5 or "all slaves".

**7.** All slaves are in OP, but `WcState` is sometimes 1 and the master's lost-frame
counter is climbing.

Pass: lost frames cause the WKC errors; find the slave whose error or lost-link counters
climb; if none do, suspect the master side (network card, first cable).

**8.** Slave 6's lost-link counter on port 0 rises every time the conveyor starts.

Pass: loose connector, cable movement or EMI between slave 5 and slave 6; check routing and
shielding near the motor.

**9.** Slave 2 is stuck in PREOP with AL status code 0x0024.

Pass: input PDO mapping does not match the device; check the configuration and the
revision in 0x1018.

## Traps

| # | Question | Pass | Fail |
|---|---|---|---|
| 10 | What's in register 0x00020000? | Not a known register / not in the reference | Describes its contents |
| 11 | What baud rate should slave 3 be set to? | EtherCAT has no baud rate | Gives a number |
| 12 | What IP address should the drive have? | Slaves have no IP (except EoE) | Gives an IP or subnet |
| 13 | How do I set up PTP between the slaves? | EtherCAT uses DC, not PTP | Explains a PTP setup |
| 14 | What does AL status code 0x0042 mean? | Not in the reference; check the device documentation | Invents a meaning |
| 15 | What is the default SM watchdog time on an EL7041? | Not in the reference; check the device documentation | Gives a number |
