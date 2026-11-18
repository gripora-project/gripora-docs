# Adjustments

The Adjustments tab lets a transmitter switch or knob change a setting live,
without reconnecting the Configurator: a gain knob, steering trim buttons,
or a profile switch, so you can try values while driving.

The functions on offer are the ones that matter on a car (the roll and
pitch versions are hidden):

| Group | Functions |
|---|---|
| Profile Selection | Battery Profile, LED Profile, Profile Selection (PID profile), Rate Profile Selection |
| Rates, RC Rates, RC Expo | Steering Rate, Steering RC Rate, Steering RC Expo |
| PID Gains | Steering P, I, D, F and B gain |
| Filters | Steering Gyro Cutoff, Steering D-term Cutoff |
| Steering Dynamics | Steering Dynamic Ceiling Gain, Dynamic Deadband Gain, Dynamic Deadband Filter |
| Driving Feel | Master Gain Steering, I-Term Decay Steering, I-Term Relax Steering (see [Profiles](profiles.md#driving-feel)) |
| Servo Trims | Servo Trim Steering (see [Servos](servos.md#trimming-from-the-transmitter)) |
| Setpoint Boost | Steering Setpoint Boost Gain |

A function under live adjustment shows its value on the tab that owns it while the switch or knob is in control.

PID and rate profile switching is a special case: each is its own function,
mapped to a channel like any other, but a profile switch applies, and is
confirmed with its own beep count, at once. The drive aids belong to the PID
profile, so switching PID profile switches them too.

Adjustments aren't saved by themselves: they are for trying values while
driving. If a value you found is worth keeping, set it on its tab (for
example [Profiles](profiles.md)) and save it there. Stepped servo trims are
the exception; see below.

The Setup Wizard's **Trim and gain** step sets up a steering trim and a gain
knob for you (Master Gain Steering on a knob, 0% fully down, 75% at center, 150%
fully up). See [Set Up Your Car](../../getting-started/car-setup.md).

## Channel names

Every channel picker in the Configurator (here, on [Mixer](mixer.md),
[Conditions](logic.md) and [Modes](auxiliary.md)) uses one naming scheme:
**CH #N**, where N is the receiver channel number, with its role where it
has one. In the car channel order AUX 1-4 are CH #3-6.

## Enable Channel and Value Channel

Each adjustment slot has two channels:

- **Enable Channel** gates whether the slot is active at all -- it only
  takes effect while this channel's value falls within the range you set
  (or set it to **ALWAYS** to skip gating entirely and let Value Channel
  control it unconditionally).
- **Value Channel** is read once the slot is enabled, and drives the
  function according to whichever mode you pick below. Its **AUTO**
  option reuses the Enable Channel itself as the value source, for the
  common case of one switch/knob doing both jobs.

## Mapped vs. Stepped

- **Mapped** continuously maps the Value Channel's position across its
  range straight onto the function's Min/Max -- a knob or 3-position
  switch gives you a proportional live value, the same idea as a
  transmitter-side gain knob but applied on the controller side.
- **Stepped** instead treats the Value Channel like a momentary
  increase/decrease button: entering one sub-range nudges the value up by
  **Step** each time, the other sub-range nudges it down -- useful for a
  2-position switch you flick repeatedly to walk a value up or down
  without needing a channel that can hold a precise proportional position.

Mapped ignores small movements of the Value Channel -- anything within about
3 µs of the position it last acted on -- so a pot that isn't perfectly still
(receiver or ADC noise, or resting right on a value boundary) doesn't flip
the value up and down by one step every loop, which would otherwise cause a
config write, a beep and a blackbox event each time and make PID values
wander. Deliberate, slow movement still gets through, because the reference
only moves once that small window is exceeded. Stepped is not affected.

For [Servo Trims](servos.md#trim) the two modes differ in more than how
you drive them: Stepped changes the servo's saved trim, one step per press,
while Mapped sets a live trim that is never saved. Neither changes the
servo's Mid, and both together are limited to 20% of the servo's Scale.
