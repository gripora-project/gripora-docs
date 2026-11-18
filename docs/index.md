<!-- redeploy: 2026-08-06T13:xxZ -->
<p align="center">
  <img src="assets/gripora_icon.svg#only-light" alt="Gripora" width="120">
  <img src="assets/gripora_icon_white.svg#only-dark" alt="Gripora" width="120">
</p>

# Gripora

**Gripora** is open-source stability-control firmware for RC cars -- a gyro
controller that keeps the car pointed where you steer it -- along with the
Gripora Configurator used to set it up.

What it does today:

- **Steering stabilization.** A yaw-rate gyro loop on the steering servo
  holds the car on the line you steer and catches slides. It corrects the
  right way in reverse and fades out near a standstill
  ([Drive State](driving/drive-state.md)), and can be switched off from the
  transmitter ([GYRO OFF](driving/gyro-off.md)).
- **Drive aids.** Timed ABS and traction control shape the brake and
  throttle you give, per PID profile ([Drive Aids](driving/drive-aids.md)).
- **A car setup out of the box.** Steering on S1, a reversible ESC on M1,
  optional rear steering and a second ESC, and the car radio channel order
  ([Car Mixer and Channels](driving/car-mixer.md)).

Launch control and sensor-based aids are planned.

Gripora builds on the work of [WingFlight](https://github.com/WingFlight),
a fixed-wing flight controller, which is itself a fork of
[Rotorflight](https://rotorflight.org), which in turn derives from
[Betaflight](https://betaflight.com). It keeps their MSP protocol, CLI, and
configuration philosophy; the aircraft features have been removed.

!!! warning "Early-stage project"
    Gripora is under active development. Firmware is currently distributed
    as numbered snapshots (`0.0.x`) rather than stable releases, and both the
    firmware and this documentation are evolving quickly. Some screenshots
    and a few labels in the apps still show the fixed-wing software Gripora
    was forked from. If something here is wrong or missing, please open an
    issue or a PR.

## Where to start

- New to Gripora? Start with [Getting Started](getting-started/index.md)
  and [Set Up Your Car](getting-started/car-setup.md).
- Looking for a Configurator tab? See the
  [Configurator](configurator/index.md) section, organized tab by tab.
- Want to understand what the controller does on the car (reverse, low
  speed, ABS, traction control, GYRO OFF)? See
  [Driving Modes & Features](driving/index.md).
- Setting up from the radio? See the [Ethos Lua Suite](ethos-suite/index.md)
  or the [EdgeTX Lua Scripts](edgetx/index.md).
- Looking for CLI settings or MSP details? See [Reference](reference/index.md).
- Want to build the firmware or configurator yourself, or contribute? See
  [Contributing](contributing/index.md).

## Getting help

Open an issue on [GitHub](https://github.com/gripora-project/gripora-firmware/issues) for setup help,
tuning advice, and project discussion.
