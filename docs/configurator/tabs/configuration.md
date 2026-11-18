# Configuration

The Configuration tab covers system-level setup that doesn't belong to a
more specific tab: board and sensor alignment, loop rates, sensors, the
ready-to-arm wiggle, features and the serial ports.

Get the board alignment right early. If the controller isn't mounted flat
and facing forward in the car, the alignment here corrects for it, so the
gyro's yaw axis is the car's vertical axis and the steering correction
works on the right axis.

## Board Alignment

Board Alignment is the coarse correction for how the controller is
physically mounted, typically in 90° steps (sideways, upside down,
rotated). Rather than working out the rotation by hand, **Auto-Align FC**
detects it for you: place the car flat and disarmed, press Start, then
briefly lift the rear by at least 20° within 15 seconds. The wizard reads
the accelerometer movement and reports the detected roll, pitch and yaw. It
needs a calibrated accelerometer and only runs while disarmed.

## Mounting Trim

Mounting Trim is a separate, fine correction (in degrees) for a mounting
surface that isn't perfectly level, applied *after* Board Alignment rather
than by changing it. Keep the two apart: Board Alignment's roll, pitch and
yaw are applied in a fixed order, so once a 90° correction is on one axis, a
small correction entered as more board alignment can end up on the wrong
axis. Mounting Trim always corrects the intended physical axis, whatever
the board alignment.

Roll and pitch mounting trim can be detected automatically: place the car
level and disarmed, press **Auto-Detect Trim** and don't touch it. Yaw
can't be measured from a resting accelerometer and stays manual. If the
remaining tilt is over 30°, Board Alignment itself is usually wrong; run
Auto-Align FC first, then try again.

Mounting Trim rotates the gyro as well as the accelerometer, so use it to
square the board up with the car, not to make the attitude display read
level on a sloping bench.

## System configuration

- **Gyro update frequency** and **PID loop frequency** set how fast the
  gyro is read and the steering loop runs. The real-time load should stay
  under 70% at the loop rate you choose.
- **Accelerometer**, **Barometer** and **Magnetometer** turn those sensors
  on or off. The accelerometer is used for the board orientation and level.
  Altitude and compass heading are not used by the control loops; they are
  only reported, for example in telemetry.
- **Ready-to-arm steering wiggle** moves the steering briefly when the car
  becomes ready to arm, as a visual confirmation. See
  [Ready-to-Arm Wiggle](../../driving/ready-to-arm-wiggle.md).

## Personalization

**Car name** names the car. It shows in the Configurator, in telemetry,
and as the prefix of Blackbox log files.

## Features

Optional features of the firmware. If a feature is off again after **Save
and Reboot**, this controller or build doesn't support it.

## Serial Ports

Each serial port (UART) gets a function and a speed, for example
**Receiver: Primary**, **Receiver: Backup 1** / **Backup 2**, **ESC:
Telemetry**, **ESC: SRXL2**, **GPS**, **Bus: S.BUS Output**, **Bus: F.BUS
Master**, **Bus: S.PORT Master**, **Bus: CRSF Sensors**, a telemetry
protocol, **Blackbox Logging** or **MSP**. Changes need **Save and Reboot**.

The CLI command `serialdebug` helps when a device on a port isn't working;
see the [CLI Command Reference](../../reference/cli-reference.md).
