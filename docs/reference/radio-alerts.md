# Radio Callouts and Alerts

The Gripora Lua suite for Ethos radios reads the controller's status over
telemetry. It tells you about important changes in two ways:

- **Spoken callouts**, such as "Backup receiver active" or "GPS not
  responding".
- **A banner along the bottom of the dashboard** that stays up for as long as
  a problem lasts.

This page explains what each one means and what to do about it.

!!! note "The suite is being converted to the car"
    The Ethos suite's alerts and callouts are being reworded for cars. The
    list below follows the alerts the firmware reports and the suite
    currently shows; some wording on the radio may still differ.

## What you need

- Telemetry working between the controller and the radio, over FrSky
  S.Port, F.Port or FBUS, or over CRSF (ELRS and Crossfire). Telemetry is on in
  new and reset configurations. For CRSF, `crsf_telemetry_mode` must be
  `CUSTOM`, which is the default; native CRSF telemetry doesn't carry the
  status sensors. See [Telemetry](telemetry.md).
- The **System Status** and **System Config** telemetry sensors selected on the
  [Receiver tab](../configurator/tabs/receiver.md). They are in the default
  sensor list, and the suite's **Default** button on the radio selects them
  too.

## The dashboard banner

When the controller reports a problem, a banner appears across the bottom of
the dashboard:

- **Red**: critical. Something is wrong now and needs your attention while
  driving.
- **Amber**: warning. Usually something to fix before you drive.

Only one banner shows at a time, the most important one. If other problems are
active too, the banner ends with a count, for example
**BACKUP RX IN CONTROL (+2)**. When the top problem clears, the next one takes
its place.

The banner stays up while the condition lasts. It isn't a pop-up that times
out, and there's nothing to dismiss.

## Status alerts

These come from the controller's status telemetry. In the table, "When it
starts" and "When it clears" are the callouts.

| Banner | When it starts | When it clears | What it means | What to do |
|---|---|---|---|---|
| **BACKUP RX IN CONTROL** (red) | "Backup receiver active" | "Main receiver restored" | The main receiver has lost its signal and the [backup receiver](../driving/backup-rx-input.md) is now driving the car. See the note below. | Bring the car back and stop. Find out why the main link dropped before the next run. |
| **FAILSAFE** (red) | (the mode callout says "Failsafe") | – | The controller is in a failsafe stage. You will rarely see this, because telemetry travels over the main link, so a link loss usually cuts telemetry too. | See [Failsafe](../configurator/tabs/failsafe.md). |
| **BATTERY CRITICAL** (red) | (the low voltage callout covers this) | – | Pack voltage is below the critical level set on the [Power tab](../configurator/tabs/power.md), or, if consumption alerts are on, the pack capacity is used up. | Stop now. |
| **GYRO OVERFLOW** (red) | "Gyro overflow" | – | The gyro has been driven past its measuring range, usually by violent vibration or a crash. The steering stabiliser can't trust its readings while this lasts. | Stop and check how the controller is mounted, and for vibration from the drivetrain. |
| **BACKUP RX NO SIGNAL** (amber) | "Backup receiver lost" | "Backup receiver OK" | A configured backup receiver isn't receiving. With two backups, this fires when either is down. If the main link drops now, part or all of the backup is missing. | Check that each backup receiver is powered and bound, and that its transmitter module is on. |
| **GPS NOT RESPONDING** (amber) | "GPS not responding" | – | The GPS was working earlier in this session and has stopped talking to the controller. This is a wiring or power fault, not a lost fix. | Check the GPS wiring and power. |
| **ACC NOT CALIBRATED** (amber) | – | – | The accelerometer has never been calibrated, so the attitude estimate uses the wrong level. | Calibrate the accelerometer with the car on a level surface, for example in the [setup wizard](../getting-started/car-setup.md). |
| **SETUP ACTIVE** (amber) | (the mode callout says "Setup") | – | A setup tool is holding the car, such as the Configurator's setup wizard or a servo, motor or mixer test override. The outputs aren't under normal control, and mode callouts pause until it ends. | Close the setup wizard or turn the override off in the Configurator. Arming is blocked meanwhile. |
| **REBOOT REQUIRED** (amber) | – | – | A setting was changed that only takes effect after a reboot. | Power cycle the controller. |
| **BLACKBOX FULL** (amber) | "Blackbox full" | – | The onboard [Blackbox](../configurator/tabs/blackbox.md) log storage is full, so this run isn't being logged. | Download the logs if you want them, then erase the flash. |

!!! note "When you'll hear the backup receiver callouts"
    Telemetry reaches the radio through the main receiver. With ELRS,
    Crossfire and FrSky, telemetry uses the same radio link as control, so
    when that link drops, the radio usually loses telemetry too. It then can't
    hear "Backup receiver active". Ethos announces the telemetry loss instead,
    and when the link comes back you'll hear "Main receiver restored".

    You'll hear "Backup receiver active" when the main receiver keeps its
    radio link but stops delivering control to the controller, for example a
    broken signal wire while S.Port telemetry uses its own wire.

    "Backup receiver lost" and "Backup receiver OK" are the ones to rely on:
    they tell you before a run whether the backup can take over.

A problem that is already present when the radio connects shows its banner,
but isn't announced. You'll hear it when it changes after that. Link changes
must hold for a moment before they're announced, so a flickering link doesn't
chatter.

## Other callouts

| Callout | When |
|---|---|
| "Armed" / "Disarmed" | The car is armed or disarmed. |
| Mode name ("Normal", "Passthrough" for GYRO OFF, "Failsafe") | The active mode changes. |
| "Setup" | A setup tool starts holding the car. Mode callouts pause until it ends. |
| "Control limit" | The steering stabiliser asked for full servo travel: it has run out of authority. Off by default, because it can be chatty. |
| "GPS Fix" / "GPS Fix Lost" | The GPS gains or loses its position fix. |
| "Profile" + number, "Rates" + number | The PID or rate profile changes. |
| "Battery" + capacity and cell count | The battery profile changes. |
| "Low Voltage" | Pack voltage is below the warning level, repeated at the set interval. |
| "Fuel" + percentage, "Low Fuel" | SmartFuel crosses a callout step, or reaches empty. |
| "ESC Temperature Warning", "BEC Voltage Warning", "RX Batt Voltage Warning" | Past the thresholds set on the radio. Each is off by default. |

## Turning callouts on and off

On the radio, open the Gripora suite and go to **Settings → Audio**. The
state callouts (armed, mode, GPS fix and profile changes) and the status
alerts have their own switches:

| Status switch | Covers | Default |
|---|---|---|
| Backup receiver | Backup receiver active / main restored, backup lost / OK | On |
| Gyro overflow | Gyro overflow | On |
| GPS not responding | GPS not responding | On |
| Blackbox full | Blackbox full | On |
| Control limit | Control limit | Off |

The switches only control the callouts. The dashboard banners always show.
