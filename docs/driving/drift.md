# DRIFT

**DRIFT** is a mode for drifting. While it is on, the steering wheel steers
the wheels directly, as with [GYRO OFF](gyro-off.md), and the gyro only
catches the car when it rotates too fast: it adds counter-steer against the
turn rate beyond an allowance. There is no heading hold.

The normal gyro loop treats the steering wheel as a turn-rate request and
steers the wheels to get that rate. In a drift that is the wrong way round:
the car rotates one way while the front wheels point the other way
(counter-steer), so you need to set the wheel angle yourself. DRIFT gives
you the wheel angle, and keeps the gyro for what it is good at in a drift:
holding the slip angle and stopping a spin.

## What it does

| | Normal | DRIFT | GYRO OFF |
|---|---|---|---|
| Steering wheel | A turn-rate request, through the rates and expo | Straight to the servo: full stick = full servo travel | Straight to the servo |
| Gyro | Steers the car to the requested turn rate | Counter-steers only the turn rate beyond **Allowed turn rate** | None |
| Steering I-term (heading hold) | Yes | Held at zero | Held |
| Steering D-term | Damps as set in the profile | Damps as set in the profile | None |
| Throttle | Through the drive aids | Unchanged: through the drive aids | Unchanged: through the drive aids |

The counter-steer grows in a straight line with the turn rate past the
allowance:

- **Counter-steer gain** 100 % gives full lock 360 °/s past the allowance.
  200 % gets there twice as soon (180 °/s past it); 0 % turns the
  counter-steer off, and DRIFT then steers like GYRO OFF (the D-term still
  damps, if the profile has any).
- **Counter-steer limit** caps it, as a share of full lock.

With the defaults (gain 100 %, allowed rate 30 °/s, limit 100 %) the gyro
leaves the car alone up to 30 °/s, and reaches full counter-lock at
390 °/s. With gain 150 % and limit 60 % it starts at 30 °/s and reaches its
60 % cap at 174 °/s. The Configurator's Drift panel shows these two rates
for the values you enter.

The counter-steer is added to the wheel angle you give, so if you already
hold full counter-lock the gyro can't add more: the total is held to full
servo travel.

## Setting it up

1. **Set the steering throw first.** In DRIFT, as in GYRO OFF, full steering
   is full servo travel. Set the end points with GYRO OFF on, as described
   in [Setting the steering throw](gyro-off.md#setting-the-steering-throw).
2. **Put DRIFT on a switch**, on the [Modes](../configurator/tabs/auxiliary.md)
   tab or in the setup wizard's **Mode switches** step. A switch you can
   reach while driving is best; a 3-position switch can carry DRIFT and
   GYRO OFF together.
3. **Check the settings** on the [Profiles](../configurator/tabs/profiles.md#drift)
   tab, in the **Drift** panel, on the Ethos Lua suite's
   [Drift page](../ethos-suite/flight-tuning.md#drift) (Tuning → Steering),
   on the EdgeTX tool's [Drift page](../edgetx/index.md#drift), or in the CLI (`drift_gain`,
   `drift_allow`, `drift_limit`). They belong to the PID profile, so a
   drift profile and a grip profile can each have their own.
4. **Bench check**, disarmed: with DRIFT on, the wheels follow the steering
   wheel. Turn the car quickly by hand around its vertical axis: past the
   allowed rate the wheels steer against the turn. (Armed with an RPM
   source, the counter-steer is faded out at a standstill, see below, so
   check it disarmed.)

Switch DRIFT on or off on a straight. The steering changes from the
stabilised command to the raw wheel angle (or back) at once, so switching
in mid-corner moves the wheels.

## Tuning

Start from the defaults and change one value at a time.

| What you see | Try |
|---|---|
| The gyro fights the drift and pulls the car straight | Raise **Allowed turn rate** |
| The car spins out before the gyro catches it | Lower **Allowed turn rate**, or raise **Counter-steer gain** |
| The counter-steer snaps the car straight, or swaps the drift to the other side | Lower **Counter-steer limit**, then the gain |
| The steering shakes in a drift | Lower the gain; check the steering D-term on the Profiles tab is not too high |

The allowed rate depends on the car and the surface: a car that drifts at a
large angle on a polished floor rotates faster than one on carpet. A
Blackbox log of a few drifts with DRIFT off shows the turn rates you
actually use; set the allowance a little above the steady rate of a good
drift.

The setup wizard's **Drift** driving style is a different thing: a set of
rates (300 °/s, 60 % expo) for the normal gyro loop. It does not turn on the
DRIFT mode.

## Interactions

- **GYRO OFF wins.** With both on, the steering is GYRO OFF: no gyro at all.
- **Failsafe wins.** While failsafe is active, DRIFT is ignored and the
  normal gyro loop steers, so a switch left on when the link dropped
  doesn't matter. See [Failsafe](../configurator/tabs/failsafe.md).
- **Drive state.** The counter-steer and the D-term are scaled by the drive
  state's gyro correction, as the normal loop's P and D are: none at a
  standstill (with an RPM source), reduced in reverse, and faded back in
  after a direction change without RPM. In reverse the gyro is negated, so
  the counter-steer still works against the rotation. See
  [Drive State](drive-state.md).
- **Gain attenuation.** The throttle and speed attenuation on the
  [Profiles](../configurator/tabs/profiles.md#gain-attenuation) tab scale
  the counter-steer and D-term. The wheel angle you give is never scaled.
  The profile's P gain and Master Gain don't change the counter-steer:
  **Counter-steer gain** alone sets it.
- **Drive aids.** DRIFT doesn't touch the throttle: ABS, traction control
  and launch control work as usual. Use
  [DRIVE AIDS OFF](drive-aids.md) to turn them off.
- **Mixer input limits apply.** Unlike GYRO OFF, DRIFT goes through the
  stabilised steering input, so a min or max set on it with the CLI
  `mixer input` command still limits the steering.
- **No heading hold.** The steering I-term is held at zero while DRIFT is
  on, so leaving DRIFT starts the normal loop clean, without a stored
  correction.
- **PID mode 0** (feedforward only, no gyro) has no gyro to counter-steer
  with; DRIFT does nothing there.

## Settings

| Setting | Configurator | Ethos | EdgeTX | Default | Range |
|---|---|---|---|---|---|
| `drift_gain` | Counter-steer gain | Counter Gain | Counter gain | 100 % | 0-200 % |
| `drift_allow` | Allowed turn rate | Allowed Rate | Allowed rate | 30 °/s | 0-720 °/s |
| `drift_limit` | Counter-steer limit | Counter Limit | Counter limit | 100 % | 10-100 % of full lock |

DRIFT is mode id 68. The settings of the profile in use are recorded in the
Blackbox log header as `drift` (gain, allowed rate, limit).

!!! warning "Racing rules"
    DRIFT uses the gyro. Check the rules of your class before running it at
    a race.

Source: `pidApplyDrift()` in `src/main/flight/pid.c` and `src/main/pg/drift.h`
in the firmware.
