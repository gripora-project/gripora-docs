# Building the Firmware

Gripora firmware is built with the same ARM GCC toolchain and Makefile
structure as Rotorflight and Betaflight. Run every command below from the root
of a [gripora-firmware](https://github.com/gripora-project/gripora-firmware)
checkout. `make help` lists all the options.

## Prerequisites

- A Linux, macOS, or WSL environment (native Windows builds of board
  firmware are not supported directly; the SITL simulator target is the
  exception, see [below](#simulator-sitl))
- `make`, `git` and `python3`

## Installing the toolchain

The ARM toolchain does not need to be installed by hand:

```
make arm_sdk_install
```

downloads the version the Makefile expects into `tools/`. Run it once.

## Targets

Gripora builds **unified targets**: one firmware per MCU family, listed in
`make/targets_list.mk`:

| Target | MCU family |
|---|---|
| `STM32F405` | STM32F405 |
| `STM32F411` | STM32F411 |
| `STM32F7X2` | STM32F722 / F7x2 |
| `STM32F7X2M` | STM32F7X2 variant with the FBUS mux FPGA |
| `STM32F745` | STM32F745 |
| `STM32G47X` | STM32G47x |
| `STM32H730` | STM32H730 |
| `STM32H743` | STM32H743 |

Board-specific pin mappings and defaults are not in the firmware repository.
They live in [gripora-targets](https://github.com/gripora-project/gripora-targets)
and are applied as a configuration on top of the unified firmware when it is
flashed. The only other target is **SITL**, below.

## Building

Build one target, for example the one preferred for test builds:

```
make TARGET=STM32F7X2
```

or every unified target, as the project's CI does:

```
make unified
```

The firmware files are written under `obj/`. Extra defines go in `OPTIONS`,
for example `make TARGET=STM32F7X2 OPTIONS="USE_SOMETHING"`; see
[Customizing the Build](tech/customizing-the-build.md).

### Version string

The version comes from `src/main/build/version.h`. CI adds a suffix with
`FC_VER_SUFFIX` (a snapshot tag, or on a feature branch the `feature/<name>`
slug plus the short git SHA). `FC_VERSION_STRING` must stay under 30 bytes,
so a long suffix fails the build with `fc_version_string_too_long`. Keep
feature branch names short. Never change the version numbers themselves in a
pull request; see the [Developer Guide](developers.md#never-bump-the-msp-api-or-firmware-version).

## Unit tests

```
make test
```

builds and runs the unit tests in `src/test/unit` natively, with no board. See
[Code Guidelines and Testing](code-guidelines.md#unit-testing).

## Flashing a locally built firmware

A locally built `.hex` file can be flashed the same way as an official build,
from the Configurator's Firmware Flasher, using **Load Firmware [Local]**
instead of the online build list. Then apply your board's configuration from
gripora-targets.

## Simulator (SITL)

`TARGET=SITL` builds Gripora as a native executable that exchanges sensor and
output packets with an external simulator over UDP, so the firmware can run
without a board. Unlike the ARM builds, it can be built natively on Windows:

```
make mingw_sdk_install    # Windows only, once: installs MinGW-w64 GCC
make TARGET=SITL          # produces obj/main/gripora_SITL.elf
```

On Linux, macOS and WSL the system `gcc` is used. RC input arrives over MSP,
and the CLI and the Configurator can connect over TCP (ports 5761 and up, one
per UART). Settings are kept in `eeprom.bin` in the working directory. A few
features that need real hardware (ADC, PWM receiver input, serial receivers,
LED strip and similar) are not built, so SITL has fewer CLI settings than a
board.

The simulator bridge and models that SITL was developed with live in the
[wingflight-sitl-hitl](https://github.com/WingFlight/wingflight-sitl-hitl)
repository and model an aircraft, not a car. See the SITL README in the
firmware repository (`src/main/target/SITL/README.md`) for ports, packets and
`eeprom.bin` handling.
