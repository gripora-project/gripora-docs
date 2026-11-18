# Failsafe

The Failsafe tab sets how the controller *detects* a lost radio link, what
value each channel takes once it has, and what the car does if the link
stays down.

Set up and bench-test failsafe *before* driving: with the wheels off the
ground, turn the transmitter off and check that the controller enters
failsafe and does what you expect.

!!! note
    If a [Backup RX Input](../../driving/backup-rx-input.md) is configured,
    losing the main link doesn't reach any of the stages below on its own:
    control switches to the backup receiver instead. The staged failsafe on
    this page only takes over if the main link and every backup link are
    down.

## Pulse Width Limit

**Minimum Pulse Width** and **Maximum Pulse Width** (default 885-2115 µs)
set the valid channel range. A channel outside it counts as failed, which
feeds the Channel Fallback below and the link-loss detection.

## Channel Fallback

What each channel does once it is judged failed:

- **Auto** sets a safe value: center for the steering, and neutral for the
  throttle. Auto is only offered for the steering and throttle channels
  (and the unused roll and pitch slots).
- **Hold** keeps the channel at its last good value. This is the default
  for every channel after the control channels, so an arm or mode switch
  that was on stays on.
- **Set** sets the channel to a value you choose, in µs.

The Configurator's help text says Auto sets the throttle "low". On a car the
firmware sets it to neutral (the Channel Center from the
[Receiver](receiver.md) tab), so the ESC neither drives nor brakes.

## What happens when the link is lost

1. **Detects the loss.** No valid frame for about 100 ms, a receiver
   failsafe flag, or a control channel outside the Pulse Width Limit counts
   as a bad signal. Arming is blocked from here on until the link is back.
2. **Holds for 300 ms.** Each channel keeps its last good value.
3. **Applies Channel Fallback (stage 1).** After 300 ms each channel takes
   its fallback value: with the defaults, steering to center and throttle
   to neutral, aux channels held. The controller keeps steering with the
   gyro at this point.
4. **Runs the Stage 2 procedure** if the link is still down after the
   guard time.

If the car was armed but the throttle had been at neutral for at least
**Failsafe Throttle Low Delay** when the link was lost, it skips the
procedure and disarms at once instead. This catches a car that was left
armed on the bench, or a transmitter switched off after the car stopped.

## Stage 2 - Failsafe Procedure

Once the guard time (**Guard time for stage 2 activation**,
`failsafe_delay`, 0.1 s units, default 1.5 s) has passed with the link
still down:

- **Drop** (the default, `DROP` in the CLI) disarms the car at once, which
  stops the motor.
- **Stop** (`AUTO-LAND` in the CLI) sends the **Failsafe throttle**
  (`failsafe_throttle`, default 1500 µs = neutral) to the ESC for **Delay
  for turning off the Motors** (`failsafe_off_delay`, 0.1 s units, default
  1 s), then disarms.

While a failsafe is active:

- The throttle is the **Failsafe throttle**, not the throttle channel's
  fallback value, and the [drive aids](../../driving/drive-aids.md) don't
  shape it. Leave it at 1500 (neutral) unless you have a reason not to; a
  value below neutral brakes (and on some ESCs reverses).
- [GYRO OFF](../../driving/gyro-off.md) and [DRIFT](../../driving/drift.md)
  are ignored, so the normal gyro loop steers even if one of those switches
  was on when the link went.
- Once the link returns, the controller needs **Failsafe Recovery Delay**
  (`failsafe_recovery_delay`) of good signal before you can arm again.

Older configurations may still hold `failsafe_procedure` 2 (GPS Rescue),
which the Configurator then shows as **GPS Rescue**. GPS Rescue no longer
exists; choose Drop or Stop.

## Setting up a safe failsafe

**The receiver's own failsafe is still your first line of defence.** It
acts faster (no 100 ms detection) and doesn't depend on the controller:

1. On your receiver or transmitter, set failsafe to steering centered and
   throttle at neutral, or to "no pulses" so the controller detects the
   loss.
2. Aux channels default to Hold on the controller side. To make a mode
   switch fall back to a known position, set that channel to **Set** with
   the value you want, or in the CLI:

   ```
   rxfail 4 s 1900
   ```

   `rxfail <channel> <a|h|s> [value]`, with the channel counted from 0. `a`
   is Auto (control channels only), `h` is Hold and `s` is Set.
3. Pick the Stage 2 procedure and check the Failsafe throttle is neutral.
4. Test it: wheels off the ground, turn the transmitter off, and check the
   wheels and the steering for the procedure you use. A receiver that keeps
   sending normal-looking frames after the link is lost can't be detected
   by the controller.

## Failsafe Test Switch

The **FAILSAFE** mode on the [Modes](auxiliary.md) tab triggers failsafe
from a switch, to bench-test your settings or as a panic switch.
**Failsafe Switch Action** picks what it does:

- **Stage 1** treats the control channels as invalid, as on a real signal
  loss: after the 300 ms hold and the guard time it runs the Stage 2
  procedure.
- **Stage 2** skips stage 1 and runs the procedure at once.
- **Kill** disarms at once.
