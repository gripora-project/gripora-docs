# Modes

The Modes tab puts modes on transmitter switches: which switch position
arms the car, turns the gyro off, turns the drive aids off, and so on.

Each mode card shows the live channel position (a marker on the slider), so
you can check a switch lands inside the range you set without driving to
find out. **Add mode** opens a searchable list of the modes, in groups.

## The modes

| Group | Mode | What it does |
|---|---|---|
| Arming & Safety | **ARM** | Arms the motor and servos. Required. The car only arms with the throttle at neutral, and arming can be blocked by other conditions; see [Status](status.md#arming-disable-flags). |
| | **PREARM** | A safety switch that must be on before you can arm. |
| | **FAILSAFE** | Triggers failsafe from a switch, for bench tests or as a panic switch; see [Failsafe](failsafe.md#failsafe-test-switch). |
| Gyro | **GYRO OFF** | Turns the gyro off: the steering wheel goes straight to the steering servo, with no gyro correction. See [GYRO OFF](../../driving/gyro-off.md). |
| | **DRIVE AIDS OFF** | Turns ABS and traction control off: the throttle and brake go to the ESC as you give them. The gyro still steers. See [Drive Aids](../../driving/drive-aids.md). |
| GPS | **GPS BEEP SATELLITE COUNT** | Beeps the number of GPS satellites in view. Only with a GPS running. |
| Beeper & Lights | **BEEPER** | Sounds the buzzer, to find the car. |
| | **BEEPER MUTE** | Silences the beeper. |
| | **LEDLOW** | Turns the LED strip off. Only with the LED strip feature on. |
| Logging & Telemetry | **BLACKBOX** | Starts and stops Blackbox logging when logging is switch-controlled. |
| | **BLACKBOX ERASE** | Erases the onboard Blackbox flash while held on. Only on boards with flash. |
| | **TELEMETRY** | Turns telemetry on for a port shared with another function. Only with the telemetry feature on. |
| Camera | **CAMERA CONTROL 1-3** | RunCam camera control. Only on builds with it. |
| User Outputs | **USER1-4** | Switch a PINIO output (a pin set up for it). Only shown when a PINIO output is linked to it. |

BEEPER, BEEPER MUTE, BLACKBOX, BLACKBOX ERASE, FAILSAFE, GPS BEEP SATELLITE
COUNT and PREARM only show in Expert Mode.

Modes that Gripora removed (ANGLE, TRAINER, ATT HOLD, GPS RTH and LOITER,
MANUAL, AUTO TRIM, THRUST VECTOR, TRADITIONAL, GOVERNOR and others) no
longer appear. Their mode ids are kept free, so the ids of the remaining
modes didn't change and existing switch assignments still work.

## Ranges and Links

A mode turns on from one of two kinds of assignment, and one mode can have
several of either:

- A **Range** watches one receiver channel: the mode is on whenever that
  channel's value is between the Min and Max you set. A 3-position switch
  can carry two modes this way.
- A **Link** turns a mode on whenever a *different* mode is on, to bundle
  modes together without a spare switch. ARM can't be linked to or from,
  and a mode that is itself defined by a link can't be linked to (no chains).

When a mode has more than one Range or Link, each one past the first gets an
**AND**/**OR** setting: the mode is on when *all* of its AND conditions are
true, or when *at least one* OR condition is true.

A switch does nothing until a mix on the radio puts it on a channel. The
Setup Wizard's **Mode switches** step puts ARM, GYRO OFF and DRIVE AIDS
OFF on switches with you and shows how to set up the radio; see
[Set Up Your Car](../../getting-started/car-setup.md).

Changes here take effect when you **Save**.
