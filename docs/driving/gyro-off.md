# GYRO OFF

**GYRO OFF** turns the steering gyro off from a switch. While it is on, the
steering wheel goes straight to the steering servo: no gyro correction, and
no rates or expo either. Full steering is full servo travel.

Put it on a switch you can reach while driving, on the
[Modes](../configurator/tabs/auxiliary.md) tab or in the setup wizard's Mode
switches step. Use it to compare the car with and without the gyro, as a
bail-out if the gyro misbehaves, and on the bench.

| | Normal | GYRO OFF |
|---|---|---|
| Steering | Stick through the rates and expo, plus the gyro correction | Raw stick, full stick = full servo travel |
| Throttle | Through the drive aids to the ESC | Unchanged: still through the drive aids |
| Steering I-term | Builds up as usual | Held: it doesn't build up while GYRO OFF is on |

- **Only the steering is bypassed.** The throttle path is the same as
  normal, so ABS and traction control keep working if they are on. To turn
  those off, use [DRIVE AIDS OFF](drive-aids.md).
- **Failsafe wins.** If failsafe starts, it takes over whatever the GYRO OFF
  switch says, so a switch left on when the link dropped can't keep driving
  the steering from a stale stick value.
- **Steering direction is the same** as with the gyro on: the raw stick is
  sent with the same sign the stabilised steering uses.
- **Mixer input limits don't apply.** GYRO OFF skips the min and max set on
  the stabilised steering input with the CLI `mixer input` command, so the
  bail-out always has the full throw. Servo **Min** / **Max** on the
  [Servos](../configurator/tabs/servos.md) tab still apply.
- Because the I-term doesn't build up while GYRO OFF is on, switching the
  gyro back on doesn't kick the steering.

GYRO OFF is mode id 12, the id WingFlight used for PASSTHROUGH, so switch
assignments carry over.

## Setting the steering throw

In GYRO OFF full stick is full servo travel, the same limit the stabilised
steering is held to. So set the steering end points with GYRO OFF on (or
with the setup wizard, which holds the steering itself):

1. With the wheels off the ground, switch GYRO OFF on.
2. Turn the steering wheel to full lock each way.
3. Adjust the servo's **Scale Neg** / **Scale Pos** on the
   [Servos](../configurator/tabs/servos.md) tab until full lock is where you
   want it, and use **Min** / **Max** to keep the servo off a mechanical
   stop.

## The setup state

SETUP is not a mode you put on a switch. It is the state the controller is
in while a setup tool holds the outputs on the bench: the Configurator's
setup wizard, or a servo, mixer or motor override. While it lasts, arming is
blocked and the Status tab shows the **SETUP** arming-disable flag. Nothing
the tool holds is saved, and if the Configurator is closed or the cable is
pulled, the controller drops it within a few seconds.

Source: `src/main/flight/mixer.c` in the firmware.
