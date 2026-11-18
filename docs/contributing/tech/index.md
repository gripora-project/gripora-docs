# Technical Reference

Internals for people working on the firmware, the Blackbox tools or the
Configurator. Most users will not need these pages.

## How the firmware is built and stored

- [Parameter Groups](parameter-groups.md) -- how settings are grouped, and the
  rules for changing them safely.
- [Configuration Format](configuration-format.md) -- how the configuration is
  laid out in flash and on the wire.
- [Atomic Barrier](atomic-barrier.md) -- how the atomic-block macros keep the
  compiler from reordering accesses.
- [Customizing the Build](customizing-the-build.md) -- building with different
  features enabled.
- [Custom Board Configuration](custom-boards.md) -- mapping pins on a custom
  board with the CLI.

## Logs and estimators

- [Blackbox Log Format](blackbox-format.md) -- the frame types, predictors and
  encodings of the log.
- [SmartFuel Internals](smartfuel.md) -- how the charge estimator works, and its
  MSP interface.

## Debugging

- [Hardware Debugging](hardware-debugging.md) -- debug adapters and how to
  use them.
- [Hardware Debugging with VS Code and J-Link](hardware-debugging-vscode-jlink.md)

## Steering control and wiring design

- [Steering Control Dynamics](flight-dynamics.md) -- the signal chain from the
  receiver to the steering servo and the ESC, sign conventions, the rationale
  behind each stage, and the known defects and limitations. Read it before
  changing the PID, mixer, drive state or drive aids code.
- [RX serial wiring auto-detect](rx-wiring-autodetect.md) and
  [ESC telemetry signalling auto-detect](esc-signaling-autodetect.md) -- the
  design of the live trial that finds the right inversion, half-duplex and
  pin-swap settings.

## Protocols

- [MSP over Telemetry](msp-over-telemetry.md) -- how the radio scripts carry MSP
  over S.Port, F.Port, FBUS and CRSF, and what limits the speed.

The firmware identifies itself over MSP as `Gripora` / `GRPA` and speaks MSP API
33.0. Gripora's own MSP2 messages are named `MSP2_GRIPORA_*` (ids `0x5Fxx`, in
`src/main/msp/msp_protocol.h`). The Configurator and both radio suites check the
identifier and accept API 33.x only, so a change to an MSP message affects all of
them. See also [MSP Overview](../../reference/msp-overview.md).
