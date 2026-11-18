# Buzzer

Gripora supports a buzzer, used for:

 * Low and critical battery alarms (when battery monitoring is on)
 * Arm and disarm tones, and warning beeps while armed with the throttle at
   neutral
 * Calibration complete
 * Beeping from a switch (the `BEEPER` mode) - useful for finding the car
   in long grass
 * Signal loss and failsafe
 * GPS fix and satellite count, when a GPS is fitted

Which events beep is set on the [Beepers tab](../configurator/tabs/beepers.md).

Three beeps shortly after power-up mean the gyro calibration has finished.
Gripora calibrates the gyro at every power-up, so keep the car still until
the beeps sound. If the car is moved during calibration, Gripora detects it
and restarts the calibration once the car is still again, which delays the
beeps. A small movement can still spoil the calibration; power-cycle the
board to calibrate again.

The CLI command `play_sound` plays the tones: repeat it to step through
them, or give it the tone's number.

## Tones

| Number | Name | When |
|---|---|---|
| 0 | `GYRO_CALIBRATED` | Gyro calibration finished |
| 1 | `RX_LOST` | Transmitter off or signal lost (repeats until the signal is back) |
| 2 | `RX_LOST_LANDING` | Failsafe AUTO-LAND is holding the throttle (SOS) |
| 3 | `DISARMING` | Disarming |
| 4 | `ARMING` | Arming |
| 5 | `ARMING_GPS_FIX` | Arming with a GPS fix |
| 6 | `ARMING_GPS_NO_FIX` | Arming with a GPS fitted but no fix |
| 7 | `BAT_CRIT_LOW` | Battery critically low (repeats) |
| 8 | `BAT_LOW` | Battery getting low (repeats) |
| 9 | `GPS_STATUS` | GPS satellite count |
| 10 | `RX_SET` | The `BEEPER` switch is on |
| 11 | `ACC_CALIBRATION` | Accelerometer calibration finished |
| 12 | `ACC_CALIBRATION_FAIL` | Accelerometer calibration failed |
| 13 | `READY_BEEP` | GPS fix found after power-up (once) |
| 14 | `MULTI_BEEPS` | A count of beeps (confirmations) |
| 15 | `DISARM_REPEAT` | Stick held in the disarm position (stick arming only) |
| 16 | `ARMED` | Armed with the throttle at neutral (repeats until the throttle moves or the car is disarmed) |
| 17 | `SYSTEM_INIT` | Power-up |
| 18 | `ON_USB` | Whether the buzzer sounds while powered from USB |
| 19 | `BLACKBOX_ERASE` | Blackbox flash erased |
| 21-22 | `CAM_CONNECTION_OPEN`, `CAM_CONNECTION_CLOSE` | Camera control connection opened or closed |

## Types of buzzer supported

The buzzer is switched by a GPIO output pin on the board, so it must make
its own tone when power is applied. Buzzers that need an analog or PWM
signal make clicking noises or no sound at all, unless the board drives the
buzzer pin with PWM (`beeper_frequency`).

## Connections

Connect a supported buzzer to the buzzer pins of your board, observing
polarity. See your board's documentation. On the bench, outside the car, you
may need to supply 5 V and ground to the board (for example through the ESC
or BEC connection) or the buzzer will not sound.
