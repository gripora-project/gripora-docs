# Configurator

This section documents the Gripora Configurator tab by tab, in the order
they appear in the menu once connected to the controller.

The tab list adapts to the connection and the firmware: some tabs only
appear when the feature behind them is set up. GPS needs a GPS port or
protocol, LED Strip the LED strip feature, XACT Servo Programming a port set
to FBUS output, and FBUS/S.Port Sensors and CRSF Sensors a port with that
function.

New to Gripora? Start with the Setup Wizard, described in
[Set Up Your Car](../getting-started/car-setup.md). Everything it sets can be
changed later on the tabs below.

| Group | Tab | Purpose |
|---|---|---|
| Setup | [Status](tabs/status.md) | Live state of the controller: arming, battery, attitude, receiver |
| | Wizard | Step-by-step car setup, see [Set Up Your Car](../getting-started/car-setup.md) |
| | [Configuration](tabs/configuration.md) | Board alignment, sensors, serial ports, features |
| | [Power](tabs/power.md) | Battery voltage and current, battery profiles, SmartFuel |
| | [Receiver](tabs/receiver.md) | Receiver protocol, channel order, telemetry |
| | [Failsafe](tabs/failsafe.md) | What happens when the radio link is lost |
| Outputs | [Mixer](tabs/mixer.md) | Model type, car setup, mixer rules, gyro direction |
| | [Servos](tabs/servos.md) | Servo center, limits, direction, trim, bus servos |
| | [Motors](tabs/motors.md) | ESC protocol, throttle range, ESC telemetry, RPM, the Drive panel |
| | [Curves](tabs/curves.md) | Mixer, gain and servo balance curves |
| Tuning | [Profiles](tabs/profiles.md) | PID profiles: Driving Feel, PID gains, gain attenuation, drive aids |
| | [Rates](tabs/rates.md) | Steering rate and expo |
| | [Gyro](tabs/gyro.md) | Gyro filtering |
| Modes & Logic | [Modes](tabs/auxiliary.md) | Modes on transmitter switches |
| | [Adjustments](tabs/adjustments.md) | Change settings live from the transmitter |
| | [Conditions](tabs/logic.md) | Logic conditions that gate mixer rules |
| Peripherals | [GPS](tabs/gps.md) | GPS module setup and status |
| | [LED Strip](tabs/led-strip.md) | Addressable LED strip |
| | [Beepers](tabs/beepers.md) | Buzzer events |
| | [Sensors](tabs/sensors.md) | Raw gyro, accelerometer and other sensor graphs |
| | [ESC Programming](tabs/esc-programming.md) | Configure the ESC through the controller |
| | [Remap FC](tabs/remap-fc.md) | Change what each pin of the board does |
| | [XACT Servo Programming](tabs/xact-servo.md) | Program FrSky XACT bus servos |
| | [FBUS/S.Port Sensors](tabs/fbus-sensors.md) | Sensors on an FBUS or S.Port master line |
| | [CRSF Sensors](tabs/crsf-sensors.md) | A CRSF sensor accessory on its own port |
| Diagnostics | [Blackbox](tabs/blackbox.md) | Data logging |
| | [Maintenance](tabs/setup.md) | Calibration, save, reset and reboot |
| | [CLI](tabs/cli.md) | Command-line access |
