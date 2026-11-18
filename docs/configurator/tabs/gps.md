# GPS

The GPS tab configures the GPS module and shows its live status (fix,
satellite count, position, speed, a map).

On a car, GPS is optional. Its speed is used by
[speed attenuation](profiles.md#speed-speed-attenuation) when the motor has
no RPM source (with an RPM source the wheel speed is used instead), and its
position and speed go to telemetry and the Blackbox. Gripora has no GPS
navigation or GPS rescue.

## When GPS is active

There is no GPS feature switch. GPS runs when the **Protocol** below matches
a port on the [Configuration](configuration.md) tab:

| Protocol | Port needed |
| --- | --- |
| NMEA or UBLOX | a port set to **GPS** |
| FBUS | a port set to **Bus: F.BUS Master** or **Bus: S.PORT Master** |
| CRSF | a port set to **Bus: CRSF Sensors** |
| MSP | none |

This tab appears as soon as any of those ports is set up, so you can pick
the matching protocol here. The **GPS BEEP SATELLITE COUNT** mode and GPS
telemetry only appear once the protocol and port match. Changes take effect
after **Save and Reboot**.

## Module setup

**Protocol** selects how the GPS module talks to the controller: NMEA or
UBLOX over a serial port, MSP (a companion device relaying GPS over MSP),
FBUS (GPS from an FrSky FBUS/S.Port sensor), or CRSF (GPS from a
[CRSF Sensors](crsf-sensors.md) accessory). For the last two, the
signal-strength panel and the baud and config options don't apply, since
both carry an already-processed fix rather than raw module output.

For NMEA and UBLOX modules, **Auto Baud** and **Auto Config** find the
module's baud rate and configure it at start-up. Leave both on unless you
have a reason to manage the module yourself. Auto Config sets a u-blox
module to 10 Hz, or 5 Hz if the module refuses 10 Hz or the port is slower
than 38400 baud. U-blox modules also have a **Ground Assistance Type**
(SBAS region: EGNOS, WAAS, MSAS, GAGAN, auto-detect or none) and **Use
Galileo** (swaps the QZSS constellation for Galileo, since a module only
tracks so many constellations at once).

Only u-blox modules report 3D speed; for the others the ground speed is
used.

**Set Home Point Once** controls when the home point
is captured (shown as Dist to Home): off (the default) resets it on every
arm; on captures it only on the first arm after power-up.

Wiring and configuring a u-blox or NMEA module by hand is covered in
[GPS Hardware and Module Setup](../../reference/gps-hardware.md).

## Diagnostics

With a serial GPS (NMEA or UBLOX) and firmware that supports it, a
**Diagnostics** panel under Status shows the GPS port and its pins, whether
it opened and at what baud rate, the GPS state, and the byte, packet, error
and timeout counters. A badge in its header gives the state at a glance
(for example *Receiving*, *No data* or *Bad data*). When something is
wrong, a line says what to do: no port set to GPS, nothing received
(wiring or power), or data arriving but no valid messages (baud rate or
protocol). In Expert Mode the panel also shows the last bytes received
from the module. For the raw data and a baud rate scan, use the CLI command
`serialdebug`; see
[GPS Hardware and Module Setup](../../reference/gps-hardware.md).
