# Car Mixer and Channels

Gripora's defaults are set up for a car out of the box: the steering servo
on **S1**, a reversible ESC on **M1**, and the channel order of a car radio.
This page describes those defaults and the one setting that fixes a gyro
that corrects the wrong way. Day-to-day setup is done on the
[Mixer](../configurator/tabs/mixer.md) tab or with the setup wizard
([Set Up Your Car](../getting-started/car-setup.md)).

## Model type

`model_type` is **Car** (`CAR`, the default) or **Custom** (`CUSTOM`).

- **Car**: the Mixer tab builds the rules from a few options and shows a
  summary.
- **Custom**: keep a mix built by hand, rule by rule.

The firmware stores only the mixer rules; the options below are how the
Configurator builds them.

## The car mixer preset

| Output | From | What it drives |
|---|---|---|
| **S1** | Stabilized Yaw (steering stick through the gyro) | Steering servo, front wheels |
| **M1** | Stabilized Throttle (signed: forward, brake and reverse) | Reversible ESC |

These are the firmware's default mixer rules:

```
mixer rule 0 set SY S1 1000 0 1000 0 0 0 0
mixer rule 1 set ST M1 1000 0 1000 0 0 0 0
```

Two options can be added on the Mixer tab (**Edit Configuration**) or in the
wizard's Model step:

- **Rear steering (4-wheel steering)**: a second steering servo on **S2**,
  also from Stabilized Yaw. **Opposite to the front wheels** gives tighter
  turns, **Same direction as the front wheels** gives crab steering.
  **Amount** (0-100 %, default 50 %) is how far the rear wheels turn compared
  with the front ones.
- **Second ESC** (dual motor, or front and rear motors) on **M2**, with the
  same throttle as M1: forward, brake and reverse.

Steering and throttle weights stay positive. If a servo turns the wrong way,
reverse it on the [Servos](../configurator/tabs/servos.md) tab; if the
motor runs the wrong way, reverse it in the ESC. The mix itself stays as it
is.

!!! warning "Feed the ESC from Stabilized Throttle"
    The throttle reaches the ESC through Stabilized Throttle. That is where
    the [drive aids](drive-aids.md) act, and where failsafe puts the throttle
    at neutral. A custom rule that feeds an ESC straight from the RC throttle
    channel bypasses both.

The throttle input is signed, -1000 to 1000 in the mixer: above zero is
forward, below zero brake and reverse. Every motor output is bidirectional
by default, so a zero command is neutral: the midpoint of `min_throttle`
and `max_throttle` (1500 µs with the defaults of 1000 and 2000).

Only steering is stabilised. The roll and pitch mixer inputs exist but are
always zero, so a rule fed from them never moves anything.

## Channel order

The default channel map is `RT1234AE`, the order of a car radio:

| Channel | Function |
|---|---|
| CH1 | Steering |
| CH2 | Throttle (trigger): forward above centre, brake and reverse below |
| CH3-CH6 | AUX1-AUX4, for mode switches, the gain knob and trims |
| CH7, CH8 | Roll and pitch, unused on a car |

On the [Receiver](../configurator/tabs/receiver.md) tab the **Car** preset
sets this order; the setup wizard can also detect the steering and
throttle channels if your radio sends them elsewhere. Reverse a channel on
the radio, not in the controller.

The throttle channel's centre (`rc_center`, default 1500 µs) is neutral.
Make sure the trigger at rest reads neutral on the Receiver tab: the car
only arms with the throttle at neutral.

## Gyro direction

With the wheels off the ground, turn the car by hand around its vertical
axis: the steering should turn **against** the movement. If it turns with
the movement, the gyro is correcting the wrong way. This usually happens
after the steering channel was reversed with a servo reverse.

Turn on **Reverse gyro (steering)** on the Mixer tab (CLI:
`yaw_gyro_reverse = ON`, default `OFF`). It reverses only the gyro feedback
of the steering loop, not the stick. Board orientation and the attitude
estimate don't change.

Don't use servo reverse to fix the gyro direction: that reverses your
steering too.

Check this driving forward only. In reverse the controller turns the gyro
correction round by itself; see [Drive State](drive-state.md). The setup
wizard's **Gyro direction** step does this check and can set it for you.

## Upgrading

The car defaults reset older settings on the first boot after an upgrade
to a firmware that introduced them: the channel map returns to `RT1234AE`,
the model type to Car and the mixer rules to the preset above. Take a
backup before upgrading, and check the mixer and channel order afterwards.
See [Backup & Restore](../getting-started/backup-and-restore.md).
