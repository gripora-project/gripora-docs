# Safety

An RC car with a powerful motor can hurt you or damage itself on the bench
as easily as on the track. Some simple rules:

* **Always** put the car on a stand with the wheels off the ground when you
  set it up for the first time, flash firmware, change the mixer, servo or
  motor settings, or if in any doubt.
* **Never** arm the car on the bench with the wheels on the ground.
* Keep fingers, cables and clothing clear of the wheels, the drivetrain and
  the steering linkage while the car is powered. The steering can move on
  its own while disarmed (the [wiggle](../driving/ready-to-arm-wiggle.md))
  and while a setup tool holds it.

## Arming

Gripora arms from an **ARM** switch, set up on the
[Modes](../configurator/tabs/auxiliary.md) tab or in the setup wizard. The
car only arms with the **throttle at neutral** (trigger at rest), not at
full brake. Use the Receiver tab to check that the throttle channel reads
neutral (1500 µs by default) with the trigger at rest, and that every
channel reaches about 1000 and 2000 µs at the ends of travel. Adjust the end
points and sub-trims on your transmitter if they don't.

If the car won't arm, the [Status](../configurator/tabs/status.md) tab and
CLI `status` list the flags that block it (for example THROTTLE when the
trigger isn't at neutral). See also
[Arming and Stick Commands](../reference/stick-commands.md).

If the car is disarmed by accident while running, after it has been armed
for at least 5 s (`rearm_min_armed_seconds`), it can be re-armed within 10 s
(`rearm_grace_seconds`) without the throttle and attitude checks, so it can
be re-armed while it is still moving.

## Failsafe

Configure failsafe in two places, and bench-test both before you drive:

1. **The receiver's own failsafe.** Set it to put the throttle at neutral
   (or to send no pulses, as your receiver offers). It is the fastest and does
   not depend on the controller at all.
2. **The controller's failsafe**, on the [Failsafe](../configurator/tabs/failsafe.md)
   tab. When the link is lost, the controller first holds the channels with
   the throttle at neutral for the guard time (`failsafe_delay`, 1.5 s by
   default), then runs its failsafe procedure: by default it disarms
   (`DROP`); `AUTO-LAND` holds `failsafe_throttle` (1500 µs, neutral) for
   `failsafe_off_delay` and then disarms. If the throttle had been at
   neutral for `failsafe_throttle_low_delay` (10 s by default) before the
   link was lost, it disarms straight away.

Once the failsafe procedure runs, the throttle to the ESC is
`failsafe_throttle`, and the drive aids and GYRO OFF are ignored.

## Check the directions

After any change to the mixer, servo reversal, ESC direction, channel order
or board orientation, check on the bench, wheels off the ground, that:

- the steering turns the right way for the steering wheel, including GYRO
  OFF if you use it;
- turning the car by hand around its vertical axis makes the steering push
  against the movement (the gyro direction; see
  [Car Mixer and Channels](../driving/car-mixer.md#gyro-direction));
- forward throttle drives the wheels forward and the brake and reverse work
  as your ESC should.

## Racing

Sensor-based driving aids, such as a steering gyro, are banned in sanctioned
racing, and class rules may ban ABS and traction control too. Check the rules
of your class before racing with Gripora.
