# Software Architecture

## Layers

```mermaid
flowchart TB
    HMI["HMI / SCADA<br/>overview · alarms · trends · setpoints"]
    subgraph PLC["PLC application"]
        direction TB
        MODE["FB_OperatingModes<br/>OFF · MANUAL · AUTO · OOS"]
        SEQ["Transfer sequence<br/>steps 0–90"]
        ALT["FB_PumpAlternation<br/>duty / standby / takeover"]
        EQ["Equipment blocks<br/>FB_Pump ×2 · FB_Valve · FB_AnalogIn ×4 · FB_Tank ×2"]
        ALM["FB_AlarmManager<br/>P1–P3 · ISA-18.2 states · first-out"]
        MAP["I/O mapping layer<br/>raw I/O ⇄ DB_Process"]
    end
    SIM["Process simulator<br/>tanks · pumps · valves · flow · pressure · faults"]
    IO["Real I/O (Siemens phase)<br/>S7-1500 + ET200SP"]

    HMI <--> MODE
    HMI <--> ALM
    MODE --> SEQ
    SEQ --> ALT
    SEQ --> EQ
    ALT --> EQ
    EQ --> ALM
    EQ <--> MAP
    MAP <--> SIM
    MAP -.-> IO
```

Control logic never touches the simulator directly: everything passes through the mapping layer, so the same
blocks run unchanged on real I/O.

## Sequence

```mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> CHECK: operator Start & demand & permissives & pump available
    CHECK --> OPEN_VALVE: OK
    CHECK --> IDLE: not OK
    OPEN_VALVE --> START_PUMP: XV101 confirmed open
    START_PUMP --> RUNNING: run feedback
    START_PUMP --> CHANGEOVER: duty failed to start
    RUNNING --> CHANGEOVER: duty equipment fault
    CHANGEOVER --> RUNNING: standby running
    RUNNING --> STOP_PUMP: demand met / operator Stop / permissive lost
    STOP_PUMP --> CLOSE_VALVE: pump stopped
    CLOSE_VALVE --> IDLE: XV101 closed
    OPEN_VALVE --> TRIPPED: travel timeout
    CHANGEOVER --> TRIPPED: standby failed
    RUNNING --> TRIPPED: protective interlock / process trip
    TRIPPED --> IDLE: reset, causes cleared
```

## Interlock classes

| Class | Applies in | Effect | Example |
|---|---|---|---|
| Protective interlock | All modes, not bypassable | Immediate stop, latched | Source low-low, leak, E-stop |
| Process permissive | AUTO | Stop at once, transfer resumes when restored | Remote permissive, reservoir level |
| Process trip | AUTO / MANUAL while running | Pump trip, station trip latched, no takeover | Dry run, high discharge pressure |
| Equipment fault | All modes | Pump fault, standby takeover in AUTO | Start failure, MCC trip |

## Platform mapping

| Concept | CODESYS (development) | Siemens (target) |
|---|---|---|
| Main cycle | `MainTask` → `PRG_Main` | OB1 |
| Startup | first-scan logic | OB100 |
| Equipment block | FB + instance | FB + instance DB / multi-instance |
| Global data | GVL `DB_Process` | Global DB `"DB_Process"` |
| Sequence / alarms | ST | SCL |
| Interlocks | ST / LD | LAD |
| HMI | CODESYS Visualization | WinCC |
