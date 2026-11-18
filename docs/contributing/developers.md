# Developer Guide

General guidance for contributing to Gripora's firmware, configurator and
radio scripts.

## Repositories

Gripora is split across several repositories under
[gripora-project](https://github.com/gripora-project):

| Repository | Purpose |
|---|---|
| [gripora-firmware](https://github.com/gripora-project/gripora-firmware) | Controller firmware (C) |
| [gripora-targets](https://github.com/gripora-project/gripora-targets) | Board configurations (custom defaults) applied on top of the unified targets |
| [gripora-configurator](https://github.com/gripora-project/gripora-configurator) | Desktop/web app for flashing and configuring the controller over MSP (Svelte/Vite) |
| [gripora-blackbox](https://github.com/gripora-project/gripora-blackbox) | Blackbox Explorer for analysing logs |
| [gripora-lua-edgetx](https://github.com/gripora-project/gripora-lua-edgetx) | Transmitter Lua scripts for EdgeTX |
| [gripora-lua-ethos-suite](https://github.com/gripora-project/gripora-lua-ethos-suite) | Lua suite for FrSky Ethos |
| [gripora-docs](https://github.com/gripora-project/gripora-docs) | This documentation site (MkDocs) |
| [gripora-website](https://github.com/gripora-project/gripora-website) | Project website and logo/branding assets |

The firmware's release and dev-build workflows also publish firmware binaries
to `gripora-artifacts`, which the web Configurator downloads from.

## Where the project stands

Gripora was forked from WingFlight, a fixed-wing firmware, and the car
conversion is in progress. Keep these in mind when changing anything:

- **Only yaw (steering) is stabilised.** Roll and pitch keep their slots in the
  per-axis arrays, PID and rate profiles, MSP messages and Blackbox fields, but
  always hold zero.
- **Throttle is signed**, -1..1 around `rc_center`: forward above neutral,
  brake or reverse below it. "Throttle off" (arming, auto-disarm, failsafe)
  means the stick at neutral.
- **Removed aircraft features keep their ids.** Attitude hold, angle/leveling,
  acro trainer, GPS rescue and navigation, thrust vectoring, the governor,
  auto trim, the tune advisor and the MANUAL and TRADITIONAL modes are gone,
  but their mode (BOX), adjustment, parameter group, debug and MSP ids are kept
  as reserved so later ids do not move. Never reuse or renumber them.
- **Remaining aircraft concepts** (for example the `WINGFLIGHT_AIRPLANE` build
  marker) are due to be replaced by car equivalents. Don't extend them, and ask
  before deleting them.
- **Car features go under `src/main/flight/`** (the directory name is
  inherited) until a reorganisation is agreed: `drive.c` (drive state),
  `drive_aids.c` (ABS and traction control), `speed_atten.c`.
- **WingFlight fixes** can be pulled in from the `upstream` remote
  (`WingFlight/wingflight-firmware`). Keep diffs to inherited code small so
  that stays easy.

## Firmware changes

See [Building the Firmware](building-the-firmware.md) to get a local build
working first, and [Code Guidelines and Testing](code-guidelines.md) before
changing steering, mixer or arming code.

### Keep stable interfaces stable

- Do not renumber permanent mode (BOX) ids. `GYRO OFF` is 12 and `DRIVE AIDS
  OFF` is 66.
- Keep MSP and CLI backward compatible where you can. If a setting is renamed,
  keep an alias where practical and record it in the firmware's `Changes.md`.

### Never bump the MSP API or firmware version

The maintainers bump these by hand when a release is made, so a pull request
must not change them:

- the MSP API version, `API_VERSION_MAJOR` / `API_VERSION_MINOR` in
  `src/main/msp/msp_protocol.h` (currently 33.0);
- the firmware version, `FC_VERSION_MAJOR` / `FC_VERSION_MINOR` /
  `FC_VERSION_PATCH_LEVEL` in `src/main/build/version.h`.

MSP changes made between releases belong to the upcoming API version. List them
in the pull request under **Compatibility**, so the release bump covers them and
the Configurator can pick them up.

The firmware identifies itself over MSP as `Gripora` / `GRPA`. The
Configurator, the Ethos suite and the EdgeTX scripts check that identifier and
the API version, so an MSP change affects all of them. Gripora-specific MSP2
messages are named `MSP2_GRIPORA_*`.

### Bump the parameter group version

Parameter groups are the exception to "don't bump versions": whenever a group
changes (fields added, removed, reordered or resized, array lengths, default
values or value ranges), bump its version, the last argument of
`PG_REGISTER*`, in the same change. Stored settings from older builds then
reset to the current defaults instead of being misread. The version is 4 bits
(0-15). See [Parameter Groups](tech/parameter-groups.md).

### Dev builds from a feature branch

Feature-branch dev builds embed `FC_VER_SUFFIX`, made from the
`feature/<name>` branch slug plus the short git SHA. `src/main/build/version.h`
requires `FC_VERSION_STRING` to stay under 30 bytes, so a long branch name makes
the workflow fail with `fc_version_string_too_long`. That is a real compile
failure, not a superseded run: keep feature branch names short (or change
`.github/workflows/dev-build.yml` to truncate the suffix).

## Configurator changes

The Configurator is a Vite + Svelte app. See its repository's README for
local dev setup (`pnpm install`, `pnpm start`). Its tabs live in `src/tabs/`
and its English text in `locales/en/messages.json`; user-facing names in these
docs follow that text.

## Docs changes

This site is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
Every page is plain Markdown under `docs/`; the site nav is defined in
`mkdocs.yml`. To preview locally:

```
pip install -r requirements.txt
mkdocs serve
```

User-facing documentation (setup, Configurator tabs, driving modes, CLI
reference) and the developer material under Contributing both live here. The
firmware repository's `docs/` holds only a pointer to this site; source code
that depends on a document links to it by URL. When a firmware pull request
changes behaviour a user sees, note in it which page here needs updating.

## Issues and pull requests

Maintainers and users read these, so keep them short and precise.

**Issues**

- Title: the symptom in one line.
- Body: firmware version and target, steps to reproduce, expected and actual
  behaviour. Attach the CLI `diff all` output or a Blackbox log when relevant.

**Pull requests**

- Title: imperative, under about 70 characters (for example "Fix steering
  I-term windup in GYRO OFF").
- Body, a few lines each:
    - **What**: the change and why it is needed. Link the issue (`Fixes #123`).
    - **Compatibility**: MSP, CLI or setting changes, and whether the
      Configurator or these docs need updating.
    - **Testing**: what was built and run (unit tests, SITL, bench, driving).
- One topic per pull request. Leave out file-by-file change lists and generated
  summaries.
