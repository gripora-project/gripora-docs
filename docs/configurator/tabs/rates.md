# Rates

The Rates tab sets the steering response: how fast the car is asked to turn
(its yaw rate) at full steering, and how much expo softens the steering
around center.

Rates are separate from the PID tune on the [Profiles](profiles.md) tab:
rates turn the steering stick into a commanded yaw rate, and the profile
decides how closely the car follows it. The Setup Wizard's **Driving
style** step sets a starting point (Beginner, Sport or Drift); see
[Set Up Your Car](../../getting-started/car-setup.md).

Up to six rate profiles can be stored and switched between from the
profile selector at the top of the page, copied from one to another, or
reset. The live curve chart and preview update immediately, so you can
compare a change before saving it. Switch rate profiles while driving with
the **Rate Profile Selection** function on the
[Adjustments](adjustments.md) tab.

## Rate, Shape and Expo

The steering (YAW) row has three values:

- **Rate** sets the top speed: the yaw rate, in °/s, the car is asked for
  at full steering. It's shown in the **Max Vel** column, and it's the
  *only* one of the three that changes it.
- **Expo** softens the response around center, for finer small corrections,
  without changing the top rate. At 0% Expo the response is a straight line
  from center to full lock.
- **Shape** sets how sharply the curve bends within the softened region:
  how much of the rate increase is saved for the last part of the steering
  travel. **Shape only does anything if Expo is above 0%.**

A car can only turn as fast as its speed and grip allow. When the stick
asks for more rate than the car can give, for example armed on a stand, the
stabilizer keeps adding steering, so held steering creeps towards full
lock. That is normal. With an RPM source, the
[Drive panel](motors.md#drive) fades the gyro out at low speed and at a
standstill, so the steering then follows the stick alone.

### Quick troubleshooting

| What you feel | Try |
|---|---|
| Not enough steering at full lock, the car won't turn tight enough | Raise Rate |
| Too much at full lock, the car spins or snaps round | Lower Rate |
| Small corrections near center feel twitchy | Raise Expo |
| Numb around center, hard to make small corrections | Lower Expo |
| Flat, linear feel all the way, no extra towards full lock | Raise Shape |
| Changing Shape does nothing | Check Expo isn't at 0% |

## Dynamics

Extra steering shaping options under **Dynamics** (expert mode):

- **Response Time** smooths the steering input before it becomes a
  setpoint. Larger values smooth more (and add lag); 0 turns it off.
- **Setpoint Boost Gain** and **Cutoff** sharpen the steering input from how
  fast the stick moves, for a snappier feel. This acts on the stick signal,
  per rate profile, before it reaches the PID loop, unlike the **B** term on
  the [Profiles](profiles.md) tab.
- **Dynamic Ceiling Gain** reduces how much stick travel is needed to reach
  the top rate while the stick moves quickly towards full lock.
- **Dynamic Deadband Gain** and **Filter** widen the steering deadband for a
  moment while the stick moves quickly back through center, to smooth a
  wobble on a fast return to center.
