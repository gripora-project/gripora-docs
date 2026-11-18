# Arming and Stick Commands

## Arming

When armed, the car is live: the ESC drives the motor as soon as the
throttle leaves neutral, and the steering gyro is active.

Arm with a switch: put the `ARM` mode on a switch on the
[Modes](../configurator/tabs/auxiliary.md) tab (the
[setup wizard](../getting-started/car-setup.md) does this). Arming needs the
**throttle at neutral**: the trigger released, within 15 µs of
`rc_center`. With the trigger pulled for throttle or brake the controller
refuses to arm and shows the `THROTTLE` flag.

By default the steering servo wiggles when the controller is ready to arm
(`wiggle_enable_ready`); see [Ready-to-Arm Wiggle](../driving/ready-to-arm-wiggle.md).

If the car is disarmed by accident while it is running, it can be armed
again with the switch for `rearm_grace_seconds` (10 s) without the throttle
and attitude checks, so it can be re-armed while still moving. This only
applies once the car has been armed for at least `rearm_min_armed_seconds`
(5 s) and the throttle had left neutral.

## Arming disable flags

If the car will not arm, open the [Status tab](../configurator/tabs/status.md),
or connect to the CLI and run `status`. The "Arming disable flags" line
lists the reasons:

| Flag | Meaning |
|---|---|
| `NOGYRO` | No gyro detected |
| `RXLOSS` | No valid receiver signal |
| `BADRX` | The arm switch was on when the receiver signal returned. Switch it off and on again |
| `BST` | The receiver signal has only just started; it must be valid for `failsafe_recovery_delay` first |
| `FAILSAFE`, `BOXFAILSAFE` | A failsafe event, or the `FAILSAFE` switch is on |
| `THROTTLE` | The throttle is not at neutral |
| `ANGLE` | The attitude estimate is not ready yet |
| `BOOTGRACE` | Still inside the power-on grace time (`pwr_on_arm_grace`) |
| `NOPREARM` | `PREARM` is set up and its switch is not on |
| `LOAD` | The system is overloaded |
| `CALIB` | Sensor calibration is running |
| `CLI` | The CLI is active |
| `MSP` | Arming has been blocked over MSP (for example by the Configurator) |
| `RPMFILTER` | The RPM filter is set up but can't run, for example because the motor has no RPM source |
| `CRASH` | Shown together with `RPMFILTER` when the motor the RPM filter needs has no RPM signal |
| `REBOOT_REQD` | A setting was changed that needs a reboot |
| `DSHOT_BBANG` | DShot bit-banging failed to start |
| `NO_ACC_CAL` | The accelerometer has not been calibrated |
| `MOTOR_PROTO` | The motor protocol is not valid |
| `SETUP` | A setup tool is holding the car: the Configurator's setup wizard, or a servo, mixer or motor override. Called `OVERRIDE` in firmware up to 0.0.32 |
| `BACKUPRX` | A [backup receiver](../driving/backup-rx-input.md) is set up and not every receiver has a link yet |
| `ARMSWITCH` | Arming was attempted while another flag was set. Switch the arm switch off, and on again once the other flags clear |

`GOVERNOR`, `GPS`, `RESCUE_SW`, `CMS` and `PARALYZE` are kept only so the
flag numbers don't move; they are never set.

## Stick commands

Stick arming and stick commands are inherited from the WingFlight
code and are **off by default** (`enable_stick_arming` and
`enable_stick_commands`).

!!! warning "Stick arming doesn't work on a car"
    The stick arming combination needs the throttle at its lowest
    position, which on a car is full brake/reverse, but arming needs the
    throttle at neutral. With `enable_stick_arming` on, the arm stick
    position is refused with the `THROTTLE` flag. Use an `ARM` switch.

The stick commands read the four control channels in the firmware's order
(roll, pitch, steering, throttle), after the channel `map`. With the default car
map `RT1234AE`, steering is CH1 and throttle CH2, and roll and pitch
are on CH7 and CH8, which a car radio often doesn't have as sticks. A
position counts as LOW or HIGH at 90% of full travel:

| Position | Approx. channel input |
|---|---|
| LOW | 1000 (throttle: full brake/reverse) |
| CENTER | 1500 |
| HIGH | 2000 (throttle: full forward) |

With `enable_stick_commands` on, these work while disarmed:

| Function | Throttle | Steering | Pitch | Roll |
|---|---|---|---|---|
| DISARM (stick arming only) | LOW | LOW | CENTER | CENTER |
| Select PID profile 1 | LOW | LOW | CENTER | LOW |
| Select PID profile 2 | LOW | LOW | HIGH | CENTER |
| Select PID profile 3 | LOW | LOW | CENTER | HIGH |
| Calibrate gyro | LOW | LOW | LOW | CENTER |
| Calibrate accelerometer | HIGH | LOW | LOW | CENTER |
| Calibrate magnetometer | HIGH | HIGH | LOW | CENTER |
| Save settings | LOW | LOW | LOW | HIGH |
| Rate profile 1 | HIGH | CENTER | HIGH | CENTER |
| Rate profile 2 | HIGH | CENTER | LOW | CENTER |
| Rate profile 3 | HIGH | CENTER | CENTER | HIGH |
| Rate profile 4 | HIGH | CENTER | CENTER | LOW |
| Disable display page cycling | LOW | CENTER | HIGH | LOW |
| Enable display page cycling | LOW | CENTER | HIGH | HIGH |

Profiles are easier to switch from the radio with an
[adjustment](../configurator/tabs/adjustments.md), or from the
[Configurator](../configurator/tabs/profiles.md).

### History

The stick commands came from MultiWii, by way of Betaflight, Rotorflight
and WingFlight.
