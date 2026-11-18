# Mixer

The Mixer tab decides what drives each output: which servo steers, which
output is the ESC, and anything else you wire up. It also holds the
steering throw, the steering invert and the gyro direction.

A rule's output is a servo (S1-S24) or a motor (M1-M4). It makes no
difference whether a servo is wired to a PWM output or is a bus servo on an
SBUS or F.Bus line: servo N is PWM output N and bus channel N alike, so one
rule drives both. See [Servos → Bus Servos](servos.md#bus-servos).

## Model Type

**Model Type** is **Car** or **Custom**. **Change Model Type** switches
between them.

- **Car** builds the rules for you from a few options and shows a summary
  instead of the rule list. This is the default.
- **Custom** shows the full rule list for a mix built by hand.

### The car setup

With **Car**, **Edit Configuration** opens the car options. These are always
set:

- **Steering servo** (front wheels) on **S1**, from the steering stick
  through the gyro (Stabilized Yaw).
- **ESC** on **M1**, reversible: forward, brake and reverse (Stabilized
  Throttle).

Two options can be added:

- **Rear steering (4-wheel steering)** puts a second steering servo on
  **S2**. **Opposite to the front wheels** gives tighter turns; **Same
  direction as the front wheels** gives crab steering. **Amount** (0-100%,
  default 50%) is how far the rear wheels turn compared with the front.
- **Second ESC** (dual motor, or front and rear motors) on **M2**. It gets
  the same throttle as M1: forward, brake and reverse.

**Apply** replaces all current mixer rules with the car set. If the board
doesn't have enough servo outputs for rear steering, the dialog says so;
assign more outputs on the [Remap FC](remap-fc.md) tab or turn rear
steering off.

The summary shows what the rules do, for example *Steering on S1, ESC on
M1*. If the rules were changed by hand and no longer match a car setup, it
says so; **Edit Configuration** rebuilds them.

If the wheels steer or drive the wrong way, reverse the servo on the
[Servos](servos.md) tab or the motor direction in the ESC. The mix stays as
it is.

The car setup names the servos it uses: `STEER`, or `F STEER` and `R STEER`
with rear steering. See [Servos → Name](servos.md#name).

More on the defaults, the channel order and the gyro direction:
[Car Mixer and Channels](../../driving/car-mixer.md).

## Steering Throw, Gyro Direction and Steering Invert

- **Steering Throw** scales every rule that reads the steering (Stabilized
  Yaw) by one percentage, 0-200% (100% is the servo's normal travel). It
  raises or lowers the throw of all steering servos together, without
  changing each rule's weight. Pair it with **Steering Override** below to
  set a target throw: enable the override, command a known percentage,
  measure the wheel angle, and adjust the throw until it matches.
- **Gyro Direction** → **Reverse gyro (steering)** reverses only the gyro's
  correction of the steering, not the stick. Turn it on if, with the wheels
  off the ground, turning the car by hand makes the steering turn *with*
  the movement instead of against it. Don't use servo reverse for this: that
  reverses your steering too. The CLI setting is `yaw_gyro_reverse`. Board
  orientation and the attitude display are not affected.
- **Steering Invert** flips every rule that reads the steering, the stick
  and the gyro correction together.

The gyro correction is turned round for reverse by the controller itself,
so Gyro Direction is set once, driving forward. See
[Drive State](../../driving/drive-state.md).

## Steering Override

**Enable steering override** forces a stabilized input to a fixed value
(-100% to 100%) for setup, for example to center or measure the steering.
It only works while disarmed, isn't saved, and is reset after a reboot.
While an override is active, arming is blocked.

## Rules

With **Custom**, the rule list is shown. Each rule reads one input and
writes it to one output, combined with what's already on that output by its
**Operator**:

- **Set** overwrites the output.
- **Add** adds to what's already there.
- **Mul** multiplies what's already there.

Rules run from top to bottom. The **first** rule for an output should
normally be **Set**, and later rules for the *same* output **Add**: a later
**Set** wipes out everything written before it. The Configurator flags both
cases on the rule.

Other fields of a rule:

- **Input source**: a stabilized input (**Stabilized Yaw** for the
  steering, **Stabilized Throttle** for the ESC), a commanded input (the
  stick after rates, without the gyro), an RC input that bypasses
  everything (**RC Steering (bypass)**, **RC Throttle (bypass)**), or any
  receiver channel, **CH #N**. The bypass and AUX inputs are labelled with
  the physical channel they come from in your channel order, for example
  `CH #1 (Steering)` or `CH #3 (AUX 1)`. Stabilized Roll and Pitch are
  always zero on a car.
- **Weight**: how strongly the input drives the output; 1000 is full.
  **Reverse** flips it.
- **Differential %** reduces (positive) or boosts (negative) the negative
  side of the rule relative to the positive side, for example to give a
  steering servo less throw one way.
- **Offset** adds a fixed amount to the output.
- **Speed** limits how fast the rule's output can move.
- **Curve** reshapes the input with a curve from the [Curves](curves.md)
  tab before the weight.
- **Condition** applies the rule only while a condition from the
  [Conditions](logic.md) tab is true; a rule that is off is dimmed.
- **Role** tags a rule, see below.

!!! warning "Feed the ESC from Stabilized Throttle"
    The [drive aids](../../driving/drive-aids.md) (ABS and traction control)
    and the failsafe throttle act on **Stabilized Throttle**. A rule that
    feeds an ESC from the RC throttle (bypass) or a raw channel skips them,
    and the ESC then doesn't get the failsafe throttle either. Stabilized
    Throttle runs from -1000 (full brake/reverse) to 1000 (full forward),
    with 0 at neutral.

### Rule roles

A **Role** is a label on a rule; the mixer itself never reads it. Two roles
exist, inherited from WingFlight: **Flap Compensation** and **Differential
Thrust Yaw**. A tagged rule's weight can be driven live from the
[Adjustments](adjustments.md) tab (**Flap Compensation Gain**,
**Differential Thrust Yaw Gain**), which scales every rule with that role,
and a negative value reverses them together. The car setup doesn't use
roles.
