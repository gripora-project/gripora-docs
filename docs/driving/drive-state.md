# Drive State

A car's yaw response to a steering angle depends on its speed and its
direction of travel. Driving backwards the response changes sign, so a gyro
correction tuned going forward would steer the wrong way and become positive
feedback. Near a standstill the car doesn't rotate at all, however far the
wheels are turned, so a gyro left fully on would wind up and slam the
steering to full lock.

The drive state is how the controller handles this. It works out which way
the car is travelling and how fast, and the steering loop uses that to:

- **turn the gyro correction round in reverse.** The steering stick keeps
  meaning "wheel angle" in both directions; only the gyro feedback is
  reversed;
- **fade the gyro correction out at low speed**, so at a standstill the
  steering follows the stick alone;
- **reduce, or switch off, the correction in reverse**;
- **clear the steering I-term** whenever the travel direction changes.

The settings are on the [Motors](../configurator/tabs/motors.md) tab, in the
**Drive** panel, which also shows the drive state live. The drive state is
also used by [ABS](drive-aids.md) to know when the ESC is braking.

## ESC behaviour

RPM sensors report how fast the motor turns, not which way. The direction
comes from what the controller tells the ESC, so it needs to know what your
ESC does with a command below neutral:

| Drive panel | CLI `drive_esc_mode` | Below neutral the ESC... |
|---|---|---|
| **Automatic** | `AUTO` (default) | Picked from the ESC protocol: **Reverse directly** with DShot (AM32's default), **Brake, then reverse** otherwise (PWM and the rest) |
| **Brake, then reverse** | `FWD_BRK_REV` | Brakes after driving forward. Back to neutral, then below neutral again, and it reverses. Most car ESCs, and AM32 with "RC car reverse" on |
| **Reverse directly** | `FWD_REV` | Reverses straight away; while the car still rolls forward it brakes or coasts first. AM32's default bidirectional mode |
| **Forward and brake only** | `FWD_BRK` | Only ever brakes. The car never reverses |

An AM32 with "RC car reverse" turned on behaves like most car ESCs and needs
**Brake, then reverse** set by hand; **Automatic** would pick **Reverse
directly** for it because it runs DShot.

If you aren't sure which your ESC does, the setup wizard's ESC step has a
test: with the wheels off the ground, hold forward, let go, then hold reverse
straight away. If the wheels only stop, and turn backwards only after you let
go and hold reverse again, it is **Brake, then reverse**.

## Speed from motor RPM

With a motor RPM source, the controller turns motor RPM into the car's
speed:

```
speed = motor RPM / gear ratio x tyre circumference
```

- **RPM sources:** bidirectional DShot RPM telemetry, ESC telemetry that
  reports RPM, or an RPM (frequency) sensor input. Set the motor's pole count
  so electrical RPM becomes motor RPM. See
  [Motors](../configurator/tabs/motors.md). DShot150 can't carry
  bidirectional DShot RPM.
- **Gear ratio:** motor turns per wheel turn, the final drive ratio. The
  Drive panel shows it as motor : wheel, so for a 10.4 : 1 final drive enter
  104 : 10. In the CLI the two numbers are the other way round, wheel first:
  `motor1_gear_ratio = 10,104`. The default is 1 : 1. The RPM filter uses it
  too, and it applies after a save and restart.
- **Tyre diameter:** outside diameter of the driven tyres,
  `drive_wheel_diameter`, 20-300 mm, default 100 mm.
- **Two motors:** motor 1's speed is used, averaged with motor 2's when
  motor 2 has an RPM source too. Each has its own gear ratio
  (`motor1_gear_ratio`, `motor2_gear_ratio`).

The car counts as **stopped** once its speed has been below 1 km/h for
0.1 s. The same speed is used by speed attenuation on the
[Profiles](../configurator/tabs/profiles.md) tab, in place of GPS speed.

Wheel speed is what is measured, not ground speed: wheelspin and locked
wheels show as faster or slower than the car is really going.

## Travel direction

**With RPM**, the direction only changes at zero speed: when the car is
stopped, or as the wheels move off from (near) zero. Braking while rolling
forward is never taken for reverse. Wheels speeding up from zero under a
command below neutral are taken as reverse (in every mode that can reverse,
since a brake can't set them turning), and under a command above neutral as
forward. So a reversal the model didn't expect, such as an ESC reversing at
its own threshold, is still caught.

**Without RPM**, the direction follows the command straight away:

- forward when the ESC is driving forward, reverse when the ESC model says it
  reverses. Braking and neutral keep the last direction;
- the ESC may brake or coast for a while before the car really changes
  direction, so after each change the gyro correction drops to zero and fades
  back in over 1 second.

!!! warning "Reverse directly without RPM"
    With **Reverse directly** (`FWD_REV`, which is also what **Automatic**
    picks on DShot) and no RPM source, the controller can't tell braking from
    reversing: any command below neutral is taken as reverse, even while the
    car is still rolling forward on the brake. The gyro correction is then
    turned round (fading in from zero over 1 s) and ABS doesn't run. Use an
    RPM source, or an ESC set to brake, then reverse.

With **Brake, then reverse** and no RPM, the controller expects reverse once
the throttle has been at neutral for at least 50 ms after a brake and then
goes below neutral again. If your ESC needs a longer pause, or a full stop,
before it reverses, the controller will think it is reversing early. An RPM
source avoids the guess.

## Steering gyro correction

The gyro's part of the steering (P, I and D) is scaled by three factors. The
feedforward (F and B), the part that comes straight from the stick, is never
scaled.

1. **Speed** (needs motor RPM). No correction below **Gyro starts at**
   (`drive_gyro_min_speed`, default 2.0 km/h), rising to full at **Gyro
   fully on at** (`drive_gyro_full_speed`, default 6.0 km/h). Without RPM
   this factor is always full.
2. **Reverse**. While reversing the correction is **Reverse gyro gain**
   (`drive_gyro_reverse_gain`, default 50%) of the forward correction, or
   none with **Gyro in reverse** off (`drive_gyro_reverse = OFF`).
3. **Direction change fade** (without RPM only): zero right after a change
   of direction, back to full after 1 s.

The steering I-term is cleared on every change of travel direction, since it
was built up for the other direction.

**While disarmed** the car is on the bench: the direction is held at
forward, the speed fade doesn't apply, and the I-term is held at zero. That
is why turning the car by hand on the bench shows the gyro correcting, which
the setup wizard's gyro direction check relies on. Every arming starts from
neutral and forward.

!!! note "Gyro direction is set once, going forward"
    The reversal for reverse is automatic. If the gyro corrects the wrong
    way, that is a forward setting: see
    [Car Mixer and Channels](car-mixer.md#gyro-direction).

## Settings

| Setting | Drive panel | Range | Default |
|---|---|---|---|
| `drive_esc_mode` | ESC behaviour | `AUTO`, `FWD_BRK_REV`, `FWD_REV`, `FWD_BRK` | `AUTO` |
| `drive_wheel_diameter` | Tyre diameter | 20-300 mm | 100 |
| `motor1_gear_ratio`, `motor2_gear_ratio` | Gear ratio (shown motor : wheel) | two numbers, wheel turns, motor turns | `1,1` |
| `drive_gyro_min_speed` | Gyro starts at | 10-1999, in 0.1 km/h | 20 (2.0 km/h) |
| `drive_gyro_full_speed` | Gyro fully on at | always above the start speed, up to 2000 (200 km/h) | 60 (6.0 km/h) |
| `drive_gyro_reverse` | Gyro in reverse | `OFF`, `ON` | `ON` |
| `drive_gyro_reverse_gain` | Reverse gyro gain | 0-100 % | 50 |

These are global settings, not per profile. The start speed can't go below
1.0 km/h, the speed at which the direction may flip, so the correction is
always faded out before the direction changes. The controller refuses a
Drive panel save as a whole if a value is out of range or the full speed
isn't above the start speed.

## Checking it

The Drive panel's **Live** box shows the ESC behaviour in use (never
Automatic: it shows what Automatic picked), the ESC state (Neutral,
Forward, Braking, Reverse), the direction, the speed and the gyro
correction in percent.

For logs, set `debug_mode = DRIVE` and record a Blackbox log. The debug
fields are: throttle command x1000, ESC state (0 neutral, 1 forward, 2
brake, 3 reverse), direction, speed x10 (km/h), speed known, correction
scale x1000, motor 1 RPM, stopped.

Source: `src/main/flight/drive.c` in the firmware.
