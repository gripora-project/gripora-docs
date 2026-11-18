# Drive Aids

The drive aids shape the throttle and brake you give before they reach the
ESC. There are two:

- **ABS** pulses the brake when you brake hard, so the wheels keep turning
  and the car keeps steering instead of locking up and sliding.
- **Traction control** limits how fast the forward throttle may rise, so
  the wheels don't spin up when you pull the trigger hard.

Both are **off by default**. Both work from the stick and time only: no wheel
speed, slip or acceleration sensor is read. Speed-based steering and
sensor-based aids are planned as separate modes.

They are set on the [Profiles](../configurator/tabs/profiles.md) tab, in the
**Drive aids** panel, from the Ethos Lua suite's ABS page, or in the CLI
(`drive_abs_*`, `drive_tc_*`).

!!! warning "Racing rules"
    Sensor-based driving aids are banned in sanctioned racing, and class
    rules may ban any electronic driving aid. Check the rules before racing
    with the drive aids, or the steering gyro, switched on.

## When the aids work

The aids only ever move the command towards neutral, never past it, and never
into the opposite direction. They pass the command through unchanged:

- while **disarmed**;
- in **failsafe** (the failsafe throttle replaces their output, so it is
  never shaped);
- while the **DRIVE AIDS OFF** mode is on. Put it on a switch on the
  [Modes](../configurator/tabs/auxiliary.md) tab to turn both aids off from
  the transmitter. The steering gyro keeps working.

[GYRO OFF](gyro-off.md) does **not** turn the drive aids off: it bypasses
only the steering gyro.

The aids act on the **Stabilized Throttle** mixer input, which is what the
car mixer preset sends to the ESC. A mixer rule that feeds an ESC straight
from the RC throttle channel bypasses them. See
[Car Mixer and Channels](car-mixer.md).

## Per PID profile

The drive aids settings belong to the **PID profile**, like the speed
attenuation settings: each of the six profiles has its own, and switching
profile (from the Profiles tab, an adjustment or the radio) switches them
with the tune. That makes a profile a track or surface preset. Copying a
profile, or resetting one, includes its drive aids settings.

## ABS

ABS works only while the ESC is **braking a car that is still rolling**:
the controller's [drive state](drive-state.md) must have the ESC in its
brake state, and, if there is an RPM source, the car must not show as
stopped. It never works in reverse or on forward throttle.

Past the **trigger** the brake is pulsed. Each pulse lasts one cycle
(1 / pulse rate):

1. **Full brake** (the brake you give) for half the cycle plus the
   **hold**, but never more than 85 % of the cycle.
2. **Release**: the brake eases to the **release brake** for the rest of the
   cycle, a percentage of the brake you give.

The release brake never drops below **10 % brake**. Most car ESCs brake
first and reverse after the throttle has been back at neutral: if ABS
released the brake to neutral, the ESC would take the next pulse as a
request to reverse. Keeping at least 10 % brake keeps the command on the
brake side of neutral, so the ESC (and the controller's drive state) stay in
brake through the whole stop. If your brake is lighter than 10 %, the
release keeps the brake you give.

Below the trigger the brake goes to the ESC unchanged, and the pulse cycle
starts again from the beginning the next time the brake passes the trigger
or after any forward throttle.

With the defaults (10 pulses per second, 20 ms hold, 40 % release brake) a
full brake is 70 ms of full brake, then 30 ms at 40 % brake, ten times a
second. The Drive aids panel shows this preview for your settings.

### Modes

| Panel | CLI `drive_abs_mode` | What it does |
|---|---|---|
| **Off** | `OFF` (default) | The brake goes to the ESC as you give it |
| **Fixed** | `STATIC` | Past the trigger the brake pulses at a fixed rate, release brake and hold |
| **Follows brake depth** | `DYNAMIC` | Like Fixed, but the harder you brake (from the trigger to full brake), the more the pulse rate moves towards the full-brake rate, the release brake eases towards 60 % of its setting, and the hold moves towards the full-brake hold |

### Steering

Two settings change the pulses with steering lock, measured from the
steering stick (not the servo): none with the wheel centred, full at full
lock.

- **Extra pulse rate** adds pulses per second at full lock, less with less
  steering.
- **Hold** (with steering) moves the hold towards this value as the steering
  approaches full lock.

The pulse rate never goes above 40 pulses per second, whatever is added.
Where a full-brake hold (**Follows brake depth**) and a steering hold both
apply, the shorter hold wins; neither can lengthen the hold beyond the base
**Hold**.

### ESC setups ABS can't run on

ABS needs to know the ESC is braking, not reversing:

- **Reverse directly** (`drive_esc_mode = FWD_REV`) **without an RPM
  source**: every command below neutral is taken as reverse, so ABS never
  runs. **Automatic** picks Reverse directly on DShot, so this includes a
  DShot ESC without RPM telemetry. The Drive aids panel warns about this.
  Fit an RPM source (bidirectional DShot, ESC telemetry or an RPM sensor), or
  set the ESC to brake, then reverse if it supports it, and set the Drive
  panel to match.
- With an RPM source, ABS stops once the car is stopped (below 1 km/h for
  0.1 s).
- **Forward and brake only** and **Brake, then reverse** ESCs work with or
  without RPM.

### ABS settings

| Panel | CLI | Range | Default |
|---|---|---|---|
| Mode | `drive_abs_mode` | `OFF`, `STATIC`, `DYNAMIC` | `OFF` |
| Trigger | `drive_abs_trigger` | 5-100 % of full brake | 30 |
| Pulse rate | `drive_abs_freq` | 2-33 pulses per second (Hz) | 10 |
| Release brake | `drive_abs_floor` | 0-100 % of your brake (never below 10 % brake) | 40 |
| Hold | `drive_abs_hold` | 0-100 ms | 20 |
| At full brake: Pulse rate | `drive_abs_dyn_freq` | 2-33 Hz (Follows brake depth only) | 15 |
| At full brake: Hold | `drive_abs_dyn_hold` | 0-100 ms, 0 = keep the hold above (Follows brake depth only) | 0 |
| With steering: Extra pulse rate | `drive_abs_corner_freq` | 0-10 Hz added at full lock, 0 = off | 0 |
| With steering: Hold | `drive_abs_corner_hold` | 0-100 ms at full lock, 0 = off | 0 |

## Traction control

| Panel | CLI `drive_tc_mode` | What it does |
|---|---|---|
| **Off** | `OFF` (default) | The throttle goes to the ESC as you give it |
| **Ramp** | `RAMP` | The forward throttle rises no faster than zero to full in the **ramp time** |

With **Ramp**, letting off is instant. After a brake or a let-off the
throttle rises again from where it is, not from where it was. Traction
control never touches the brake or reverse, nor a command above neutral
that the ESC uses to brake a car still rolling backwards.

Turning traction control on, or switching DRIVE AIDS OFF back off, while
the trigger is held never cuts the throttle: while traction control or the
aids are off, its limit follows the throttle.

| Panel | CLI | Range | Default |
|---|---|---|---|
| Mode | `drive_tc_mode` | `OFF`, `RAMP` | `OFF` |
| Ramp time | `drive_tc_ramp_time` | 50-2000 ms, zero to full throttle | 500 |

A longer ramp time is gentler: try longer on a slippery surface, shorter on
high grip.

## Checking it

The Drive aids panel's **Live** box shows whether the aids are on, what ABS
is doing (Idle, Braking past the trigger, Releasing the brake), what
traction control is doing (Idle, On while driving forward, Holding the
throttle back), and the throttle from the transmitter next to the throttle
sent to the ESC.

For logs, set `debug_mode = DRIVE_AIDS`. The debug fields are:

| Field | Value |
|---|---|
| 0 | Throttle from the transmitter, x1000 |
| 1 | Throttle to the ESC, x1000 |
| 2 | Flags: bit 0 aids on, bit 1 ABS braking past the trigger, bit 2 ABS releasing, bit 3 traction control on while driving forward, bit 4 traction control holding the throttle back |
| 3 | While braking with ABS: pulse rate x10. While driving forward with traction control: the throttle it allows, x1000 |
| 4 | ABS release brake share, x1000 |
| 5 | ABS hold, ms |
| 6 | ABS position in the pulse cycle, ms |
| 7 | Steering stick, x1000 |

Blackbox logs also carry the settings of the profile in use in the
`drive_abs` and `drive_tc` header lines.

Source: `src/main/flight/drive_aids.c` in the firmware.
