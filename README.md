# Water Pumping Station — PLC Control Application

A simulated duty/standby water transfer station, engineered the way a real one would be: control philosophy
first, fail-safe I/O, interlocks, alarm management to ISA-18.2, test cases traced to requirements — then code.
Built in CODESYS (IEC 61131-3 Structured Text) and migrated to **Siemens S7-1500 / TIA Portal / WinCC**.

> **Status:** 🟡 Phase A — Walking skeleton · Control philosophy rev A complete
> This repository shows the outcome: design, architecture, screenshots, test results. The PLC source is kept
> in a private engineering repository.

## The process

```
   TK100 source ──▶ P101 / P102 (duty / standby) ──▶ FT101 · PT101 ──▶ FCV101 ──▶ XV101 ──▶ TK200 reservoir ──▶ demand
   LT100, LSLL100                                                                          LT200, LSHH200
```

Two fixed-speed pumps transfer water on reservoir demand. The PLC runs the start/stop sequence with feedback
confirmation at each step, alternates duty, hands over to the standby pump on equipment faults (but not on
process faults, where the standby would fail the same way), and protects the station against dry run, dead-head,
low-low / high-high level, leaks and E-stop.

## What it demonstrates

| Area | How |
|---|---|
| Control philosophy | Modes, bumpless transfer, step sequence with timeouts, hysteresis, anti-short-cycling |
| Protection | Protective interlocks active in every mode; fail-safe (1 = healthy) inputs; latched trips with first-out |
| Alarm management | P1–P3 priorities, ISA-18.2 states, acknowledge separated from reset, reset refused while cause active |
| Software architecture | Reusable FBs (pump, valve, analog input, alternation, alarms), I/O mapping layer, Siemens-portable subset |
| Simulation | In-PLC process model with time acceleration and fault injection for every failure the test plan covers |
| Verification | 51 test cases and design reviews, every one of the 34 requirements covered, re-run on PLCSIM after migration |
| Siemens | FBs/instance DBs in SCL, interlocks in LAD, WinCC HMI, S7-1500 hardware configuration |

## Architecture

See [docs/architecture.md](docs/architecture.md).

## Roadmap

| Phase | Content | Status |
|---|---|---|
| A | Engineering documents, simulator, first manual control | 🟡 In progress |
| B | Equipment function blocks with unit tests | ⚪ |
| C | Automatic sequence, interlocks, takeover | ⚪ |
| D | Analog validation, flow PID, full alarm set | ⚪ |
| E | HMI: overview, alarms, trends, statistics | ⚪ |
| F | Integration and fault testing — test report | ⚪ |
| G | Siemens S7-1500 / TIA Portal migration and validation | ⚪ |

## Safety note

This is a learning and simulation project. A simulated status bit is not a safety function. Real installations
require proper electrical, motor-protection and functional-safety engineering, commissioning and validation by
qualified personnel.

## Author

Kidane Berhane — Instrumentation & Control Engineer (7 years).
