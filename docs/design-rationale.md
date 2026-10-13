# Design rationale — the short version

Thirteen design decisions, each with the question behind it and the reason for the choice.
The full version, with options and tables, is part of the functional design specification.

| # | Question | Decision | Why |
|---|---|---|---|
| 1 | How should protective inputs be wired? | 1 = healthy (NC contacts) for Stop, E-stop, level switches, leak and drive healthy | A broken wire then stops the plant and raises an alarm, instead of silently disabling the protection |
| 2 | Does the PLC need the MCC Local/Remote position? | Yes; a pump in Local is not available | No false start failures; a takeover never makes two pumps run; the HMI shows why AUTO ignores a pump |
| 3 | A station-wide Maintenance mode? | No: Out of Service per pump and valve | Maintenance on P101 should not stop AUTO on P102; OOS equipment does not flood the alarm list |
| 4 | Does selecting AUTO or MANUAL start anything? | No; AUTO waits for an operator Start | Equipment moves only after a deliberate action, never as a side effect of a selector |
| 5 | AUTO → MANUAL with a pump running? | Bumpless, including speed (86 % stays 86 %) | A mode change selects who is in control; it should not change the process |
| 6 | Should the standby always take over? | Only on equipment faults; one 10 s attempt on low flow with a healthy source; never on process faults | Both pumps share the suction and discharge: an empty tank or closed valve would damage the standby too |
| 7 | Where does the source tank get its water? | Inlet valve XV100 on level (30 % / 80 %), independent high-high switch LSHH100, low and low-low level alarms | Realistic design, and the level alarms can be tested by switching the supply off or raising demand |
| 8 | Throttling valve or variable speed drives? | VFDs, minimum speed 50 %, ramps, speed supervision | About 37 % less hydraulic power at 25 m³/h; soft starts reduce water hammer and wear |
| 9 | Does an interrupted transfer resume? | Yes: the demand latch survives stops and trips | The station finishes the job it started instead of waiting for the reservoir to drop to 40 % |
| 10 | A pump fails to stop: close the discharge valve? | No: keep XV101 open, cross-check actual speed, escalate to P1 at reservoir high-high | Dead-heading damages the pump in minutes; a filling reservoir gives the operator time |
| 11 | Flow or pressure transmitter faulted: stop the pump? | No: the other instrument backs it up; stop only if both are faulted | Dry run and dead-head each show on both instruments, so one fault does not leave the pump unprotected |
| 12 | HMI communication lost? | P2 alarm; AUTO continues; MANUAL equipment stops | In MANUAL the operator can no longer see or stop what they started; local Stop and E-stop still work |
| 13 | Are the default levels and timers right? | Yes, as simulation defaults, adjustable within limits | The order matters: switches act beyond control levels, alarms warn before trips, timeouts exceed the action they supervise |
