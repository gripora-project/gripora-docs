# MSP Overview

MSP (MultiWii Serial Protocol) is the binary protocol the Configurator and
the radio Lua scripts use to talk to Gripora firmware -- over USB or a
serial port, or through the receiver's telemetry link -- to read sensor
data, read and write configuration, and issue commands.

On connection, the Configurator queries the firmware's API version, variant
identifier (`GRPA`, reported as Gripora) and build/firmware version to
confirm compatibility before opening the full configuration UI. If a
connected firmware doesn't identify as Gripora, or reports an API version
this Configurator build doesn't support, the Configurator falls back to
[CLI-only mode](../getting-started/first-connection.md) rather than assuming
MSP compatibility it can't verify.

## Version compatibility

Gripora's MSP protocol is its own from **MSP API 33.0**: earlier Gripora
firmware reported the API version of WingFlight, which it was forked from
(22.x), and WingFlight stays on 22.x. The Gripora-specific MSP2 messages
are named `MSP2_GRIPORA_*` (ids `0x5F00` and up; they were `MSP2_WING_*`
with the same ids and payloads). The Configurator and the Lua scripts accept
MSP API 33.x only, and refuse 22.x firmware: update it.

Additive changes, such as a new read-only command, don't change the API
version: firmware that lacks a command answers it as unsupported, which is
how a client tells. Appending fields to the end of a reply is additive too,
and a client tells from the reply length. For example,
`MSP_MIXER_CONFIG` carries `yaw_gyro_reverse` as an optional trailing byte,
and `MSP2_GRIPORA_DRIVE_AIDS_CONFIG` grew from 9 to 12 bytes when traction
control was added.

Changes to an existing message's layout bump the API version, because an
old client would misparse the reply or write shifted bytes into a profile.
Use the Configurator and Lua scripts from the same release as the firmware.

## Car messages

Added for the car, all in MSP API 33.0:

| Message | Id | Content |
|---|---|---|
| `MSP2_GRIPORA_DRIVE_CONFIG` / `MSP2_GRIPORA_SET_DRIVE_CONFIG` | `0x5F3C` / `0x5F3D` | The [drive state](../driving/drive-state.md) settings: ESC mode, wheel diameter, gyro start and full speed, gyro in reverse and its gain |
| `MSP2_GRIPORA_DRIVE_STATUS` | `0x5F3E` | ESC mode in use, commanded ESC state, travel direction, whether the speed is known, speed and gyro correction scale |
| `MSP2_GRIPORA_DRIVE_AIDS_CONFIG` / `MSP2_GRIPORA_SET_DRIVE_AIDS_CONFIG` | `0x5F3F` / `0x5F40` | The [drive aids](../driving/drive-aids.md) of the current PID profile: the nine ABS settings, then the traction control mode and ramp time |
| `MSP2_GRIPORA_DRIVE_AIDS_STATUS` | `0x5F41` | Flags (aids on, ABS armed, ABS releasing, traction control armed, traction control holding back), throttle in and throttle out |

A `SET` request for the drive or drive aids settings is refused as a whole
if any value is out of range. `MSP_SET_MIXER_CONFIG` accepts only the car
model types (`CAR`, `CUSTOM`).

## Removed messages

Messages for removed features answer as unsupported. Their ids are not
reused:

- Governor: `MSP2_GRIPORA_GOVERNOR_CONFIG`, `MSP2_GRIPORA_SET_GOVERNOR_CONFIG`
- Thrust vectoring: `MSP2_GRIPORA_TV_PID_CONFIG`,
  `MSP2_GRIPORA_SET_TV_PID_CONFIG`, `MSP2_GRIPORA_SELECT_TV_PROFILE`,
  `MSP2_GRIPORA_COPY_TV_PID_PROFILE`, `MSP2_GRIPORA_TV_EFFECTIVE_PID_GAINS`
- GPS rescue and navigation: `MSP_GPS_RESCUE`, `MSP_SET_GPS_RESCUE`,
  `MSP_GPS_RESCUE_PIDS`, `MSP_SET_GPS_RESCUE_PIDS`,
  `MSP2_GRIPORA_GPS_NAV_CONFIG`, `MSP2_GRIPORA_SET_GPS_NAV_CONFIG`
- Tune advisor: `MSP2_GRIPORA_TUNE_ADVISOR`, `MSP2_GRIPORA_TUNE_ADVISOR_CLEAR`

Every other message keeps its layout. Fields of removed features inside a
message that stays, such as the angle, trainer, attitude hold and relax
fields of `MSP_PID_PROFILE`, are zero placeholders. Commands for features
removed earlier (VTX, camera control, the rangefinder) are gone too.

The mode list sent over MSP keeps the permanent ids of removed modes
reserved, so `GYRO OFF` is still id 12; `DRIVE AIDS OFF` is id 66.

!!! note
    A full MSP command reference is planned as the project stabilises --
    contributions welcome, see [Contributing](../contributing/index.md). The
    message ids and payload comments are in the firmware's
    `src/main/msp/msp_protocol.h`, and the changes per API version in its
    `Changes.md`.
