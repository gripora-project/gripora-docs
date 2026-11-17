<!-- redeploy: 2026-08-06T13:xxZ -->
<p align="center">
  <img src="assets/gripora_icon.svg#only-light" alt="Gripora" width="120">
  <img src="assets/gripora_icon_white.svg#only-dark" alt="Gripora" width="120">
</p>

# Gripora

**Gripora** is open-source stability-control firmware for RC cars -- a gyro
controller that keeps the car pointed where you steer it -- along with the
Gripora Configurator used to set it up. The planned feature set covers
steering stabilization, ABS braking, traction control, and launch control.

!!! warning "Gripora was just forked from WingFlight"
    Gripora was forked from [WingFlight](https://github.com/WingFlight), a
    fixed-wing flight controller, and the conversion to cars has only just
    started. **The documentation below still describes the fixed-wing
    functionality Gripora inherited from WingFlight** -- flight modes, control
    surfaces, aircraft setup and so on. It will be rewritten as the car
    features land. Until then, treat it as a description of the inherited
    code, not of what Gripora will do on a car.

Gripora builds on the work of [WingFlight](https://github.com/WingFlight),
which is itself a fork of [Rotorflight](https://rotorflight.org), which in
turn derives from [Betaflight](https://betaflight.com). It keeps their MSP
protocol, CLI, and configuration philosophy.

!!! warning "Early-stage project"
    Gripora is under active development. Firmware is currently distributed
    as numbered snapshots (`0.0.x`) rather than stable releases, and both the
    firmware and this documentation are evolving quickly. If something here is
    wrong or missing, please open an issue or a PR.

## Where to start

- New to Gripora? Start with [Getting Started](getting-started/index.md).
- Setting up the app from a fresh install? See the
  [Configurator](configurator/index.md) section, organized tab-by-tab.
- Want to understand a specific mode or feature (attitude hold, auto-trim, the
  throttle governor, etc.)? See [Flight Modes & Features](flight-modes/index.md).
- Looking for CLI commands or MSP details? See [Reference](reference/index.md).
- Want to build the firmware or configurator yourself, or contribute? See
  [Contributing](contributing/index.md).

## Getting help

Join the [WingFlight Discord](https://discord.gg/aEyyAJTXRw/) for setup help,
tuning advice, and project discussion.
