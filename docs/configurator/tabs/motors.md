# Motors

The Motors tab sets how the controller drives the ESC: the throttle
protocol and range, ESC telemetry, the motor RPM source, and the **Drive**
panel that tells the steering gyro which way and how fast the car is
going. It also has a **Throttle Override** to test the motor from the bench.

!!! danger "Wheels off the ground"
    Put the car on a stand with the wheels off the ground before you use
    Throttle Override or change the throttle range. Throttle Override makes
    the motor run.

## Reversible ESCs

Every motor output is reversible (bidirectional) by default: the throttle
runs from -100% (full brake or reverse) through 0 (neutral) to +100% (full
forward). With PWM, neutral is the midpoint of **Low Throttle** and **High
Throttle**: 1500 µs with the defaults of 1000 and 2000 µs. A stopped or
disarmed motor gets neutral, never the bottom of the range, which a car ESC
would read as full brake or reverse.

Calibrate the ESC to the controller with the throttle at neutral, as its
manual describes, so the car neither creeps nor brakes at neutral. The
Setup Wizard's **ESC** step walks you through it; see
[Set Up Your Car](../../getting-started/car-setup.md).

## Throttle

- **Throttle Protocol**: older and most car ESCs take PWM; others take a
  digital protocol like DShot, which needs no range calibration. Check what
  your ESC supports before you switch. SRXL2 and Castle Link are described
  below.
- **Unsyncronised ESC update** and **Update Frequency**: with PWM, the ESC
  signal runs at its own rate rather than with the steering loop. The
  default is 50 Hz; use a higher rate only if your ESC supports it.

### Throttle Range (PWM)

**Low Throttle** (`min_throttle`, default 1000 µs) is full brake/reverse,
**High Throttle** (`max_throttle`, default 2000 µs) is full forward, and
neutral is halfway between them. **Motor Off** (`min_command`) is not used
for a reversible output.

Get these wrong and the motor can run with the throttle at neutral, even
disarmed. Use **Live Update**, which applies changes at once, with Throttle
Override to check the result before trusting the values.

## SRXL2 ESC

Spektrum SRXL2 ESCs (Smart ESCs) take their throttle and send their
telemetry on one signal wire, which goes to a **serial port**, not to the
motor output.

To use one, set a spare UART to **ESC: SRXL2** on the
[Configuration](configuration.md) tab. That's all that's needed:
**Throttle Protocol**, **Telemetry Protocol** and **Half-Duplex** are then
set to match and locked (shown disabled, with a tooltip saying why). The
firmware forces the same values on save whatever is shown here. To go back
to another ESC protocol, change the port's function on the Configuration
tab.

The protocol list only offers SRXL2 once a port has that function, since
without one the ESC gets no signal. The Setup Wizard's ESC step does the
port assignment for you.

With an SRXL2 ESC there are two more settings:

- **SRXL2 Telemetry Interval**: every how many throttle frames the ESC is
  asked for telemetry. 0 turns telemetry polling off. Default 7.
- **SRXL2 Motor 2 Channel**: for dual-motor Smart ESCs (e.g. the Spektrum
  SPMXAE1020), the SRXL2 channel that carries the second motor, 2 to 32.
  Motor 1 is always channel 1. With a channel set, the motor count becomes 2
  and the mixer's second motor output (M2) drives it: turn on **Second
  ESC** in the [Mixer](mixer.md) car setup, or add a rule for M2. Leave it
  **Off** for a single motor (the default). CLI: `srxl2esc_motor2_channel`.

Both take effect after **Save and Reboot**. Dual-motor Smart ESCs report no
RPM, so RPM filtering and the speed from RPM are not available with them.

## Castle Link

Castle Link uses the motor output, like PWM. The ESC's telemetry comes back
on the same wire, so it needs no serial port and no telemetry protocol. See
[Castle ESC Telemetry](../../reference/castle-esc-telemetry.md).

## ESC Telemetry

**Telemetry Protocol** reads voltage, current, temperature, consumption and
RPM from an ESC that sends telemetry. Connect the ESC's telemetry wire to
the controller and set its port to **ESC: Telemetry** on the
[Configuration](configuration.md) tab. **Half-Duplex** and **Pin Swap** set
the wiring; **Detect Wiring** tries each combination until a valid frame
arrives (save first, ESC powered, car disarmed). **Sensor Correction**
scales the voltage, current and consumption the ESC reports.

With **FrSky F.BUS** as the protocol, telemetry comes over the F.Bus port
instead; see [FBUS/S.Port Sensors](fbus-sensors.md).

## RPM

Motor RPM is what lets the controller know the car's speed and when it has
stopped. It comes from **Dshot RPM Telemetry** (bidirectional DShot, with a
compatible ESC such as AM32, Bluejay or BLHeli32 32.7 or later), from ESC
telemetry, or from the **RPM Sensor** (frequency) input. Set the motor's
**pole count** (the number of magnets on the motor bell, not the stator
teeth) so electrical RPM becomes motor RPM.

The same RPM feeds the RPM filter on the [Gyro](gyro.md) tab, which needs a
fast source: bidirectional DShot or the RPM sensor input.

## Drive

The **Drive** panel sets what the controller needs to work out which way the
car is travelling and how fast, from the throttle it sends to the ESC and
the motor RPM. It uses this to keep the steering gyro correcting the right
way in reverse, and to fade the gyro out when the car is nearly stopped.

- **ESC behaviour**: what your ESC does below neutral. **Automatic** picks
  from the protocol: with DShot the ESC reverses directly, as AM32 does by
  default; with PWM it brakes first and reverses after the throttle returns
  to neutral, like most car ESCs. The others are **Brake, then reverse**,
  **Reverse directly** and **Forward and brake only**. An AM32 with RC car
  reverse turned on needs **Brake, then reverse**.
- **Speed from motor RPM**: **Tyre diameter** (outside diameter of the
  driven tyres, default 100 mm) and **Gear ratio** (motor turns per wheel
  turn, the final drive ratio: for 10.4 : 1 enter 104 : 10), per motor. The
  panel shows the resulting formula. Speed attenuation on the
  [Profiles](profiles.md) tab uses this speed too, in place of GPS speed.
  The gear ratio is also used by the RPM filter and applies after the save
  and restart.
- **Steering gyro**: **Gyro starts at** and **Gyro fully on at** (default
  2.0 and 6.0 km/h). Below the start speed the gyro doesn't correct the
  steering, so at a standstill the steering follows the wheel alone; the
  correction rises to full at the second speed. Both need motor RPM: without
  it the gyro is always fully on. **Gyro in reverse** (on by default) and
  **Reverse gyro gain** (default 50%) set the correction while reversing.
- **Live** shows the ESC behaviour in use, the ESC state (Neutral, Forward,
  Braking, Reverse), the direction, the speed and the gyro correction.

If no RPM source is set up, the panel says so: the controller can't tell
when the car has stopped, so the gyro stays fully active at a standstill.

How the drive state works, and what each setting does in detail:
[Drive State](../../driving/drive-state.md).

## Throttle Override

**Throttle Override** sets the throttle output to a value for setup and
testing: -100% to +100% on a reversible output. It only works while
disarmed, isn't saved, and resets after a reboot. While it is on, arming is
blocked.
