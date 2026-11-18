# CLI Command Reference

The CLI provides direct, text-based access to the controller,
available from the [CLI tab](../configurator/tabs/cli.md) in the
Configurator, or over any serial terminal at the configured baud rate.

## Common commands

| Command | Description |
|---|---|
| `help` | List available commands |
| `status` | Show system status |
| `version` | Show firmware version info |
| `dump` | Print the full configuration (including defaults) |
| `dump all` | Print the full configuration for all profiles |
| `diff` | Print only settings that differ from defaults |
| `diff all` | Print changed settings across all profiles |
| `save` | Save changes and reboot |
| `exit` | Exit the CLI (without saving) |
| `defaults` | Reset to firmware defaults |

## Backup and restore

`diff all` is the preferred backup format for day-to-day use -- compact, and
easy to review. Paste the saved output back into the CLI and run `save` to
restore it. See [Backup & Restore](../getting-started/backup-and-restore.md)
for the full workflow.

## Serial port diagnostics

`serialdebug` helps when a device on a serial port (GPS, receiver,
telemetry, ESC sensor) isn't working. It shows how each port is set up,
what the function on a port reports, and the raw data a port receives.

| Command | Description |
|---|---|
| `serialdebug` | Show every port |
| `serialdebug <port>` | One port in detail, with what its function reports |
| `serialdebug monitor <port> [<baud>] [<seconds>] [inverted]` | Print what a port receives |
| `serialdebug scan <port> [inverted]` | Try common baud rates and look for known protocols |

`<port>` is the port number used by the `serial` command (UART1 is 0,
UART7 is 6, UART10 is 9) or its name, such as `UART7`. `serialdebug` with
no arguments lists the numbers, and `serialdebug help` prints the usage.

### All ports

```
# serialdebug
port     id  rx   tx   functions         open  baud
VCP      20  -    -    MSP               yes
UART1     0  -    -    -                 no pins
...
UART3     2  D09  D08  RX_SERIAL         yes   460800 (irq rx)
UART7     6  E07  E08  GPS               yes   57600
UART10    9  E02  E03  -                 no
...
```

- **rx / tx**: the pins assigned with `resource SERIAL_RX` / `SERIAL_TX`.
- **open**: whether the port is running. `NO` (in capitals) means it has a
  function but did not open. `no pins` means no pins are assigned, so the
  port can't be used. `PIN CONFLICT` means one of its pins is also used by
  something else, such as a servo or LED; see `resource show`.
- **(irq rx)**: the port's owner reads incoming bytes as they arrive (for
  example a serial receiver); `monitor` still works on it.

Lines starting with `#` point out setup problems: a port whose function did
not open, a pin conflict, or a `gps_provider` with no matching port (for
example `UBLOX` with no port set to GPS, so GPS is off).

### One port

`serialdebug 6` shows UART7 in detail: its pins and functions, the baud
rates configured for each function type, and if it is open, the live baud
rate, direction, inversion, half duplex and how many bytes are waiting to
be read. Then each function on the port that can report its state adds a
block. GPS reports:

```
GPS: provider UBLOX, active yes
GPS: state RECEIVING_DATA, last valid message 85ms ago, at 57600 baud
GPS: bytes 91230, packets 1520, errors 0, timeouts 0, sats 11, fix yes
```

- **active**: whether GPS was started at the last boot. GPS starts only
  when `gps_provider` matches a port (see the
  [GPS tab](../configurator/tabs/gps.md)). `setup changed: save and reboot`
  means the port or provider changed since the last boot.
- **state**: `INITIALIZING` or `LOST_COMMUNICATION` means no valid messages
  are arriving. `RECEIVING_DATA` means the module is talking.
- **bytes / packets / errors / timeouts**: everything received, valid
  messages, bad messages (wrong checksum, usually noise or a wrong baud
  rate) and times the module went silent.

The hints tell the cases apart: no bytes at all points at wiring or power,
bytes without packets at the baud rate or protocol, and GPS set on more
than one port means only the first one is used.

### Watching a port

`serialdebug monitor 6` prints what UART7 receives for 5 seconds. Readable
text is shown as is, other bytes as `<hex>`. A working NMEA GPS shows lines
like `$GNGGA,...`, a u-blox in binary mode shows `<B5>b...`, a CRSF
receiver shows `<C8><18><16>...`. If nothing arrives, check that the
device's TX goes to the controller's RX pin, and check power and
ground.

The baud rate defaults to the port's current rate (or its GPS baud rate if
the port is closed). Give one to try another, for example
`serialdebug monitor 6 9600 10` listens at 9600 baud for 10 seconds (up to
60). Add `inverted` for a signal that is inverted, such as S.Bus; a port
that is already open keeps its own setting.

### Finding the baud rate and protocol

`serialdebug scan 6` listens for 1.5 seconds at each of 9600, 19200,
38400, 57600, 100000, 115200, 230400, 400000, 416666, 420000 and 460800
baud, and counts the NMEA, UBX, CRSF and FBUS frames it recognises at each:

```
   9600:   960 bytes, 100% text, 16 NMEA
  19200:   480 bytes,  41% text
...
# looks like NMEA at 9600 baud
```

Set the port's baud rate (or for GPS, turn on `gps_auto_baud`) to match.
S.Bus is not recognised; use `monitor` with `inverted` for it.

!!! note
    While `monitor` or `scan` runs the controller does nothing
    else, so only use them on the bench. Press any key to stop early. A
    port already in use (such as the GPS port) is borrowed and set back
    afterwards.

## Settings

`set <name> = <value>` changes a setting, `get <name>` shows its value and
allowed range (`get` with part of a name lists every setting that matches),
and `save` stores the changes and reboots.

The tables below list every setting in the firmware, grouped by topic.

- **Scope**: *global* settings apply whatever profile is active. *PID
  profile* settings belong to the PID profile selected with `profile <n>`
  (0-5), *rate profile* settings to the one selected with `rateprofile <n>`
  (0-5): select the profile first, then `set`. `diff all` and `dump all`
  list every profile. *global (hw)* settings describe the board (buses,
  sensor orientation, pins); they normally come from the board
  configuration and are listed by `dump hardware`.
- **Default**: the value on a unified target before a board configuration
  ([gripora-targets](https://github.com/gripora-project/gripora-targets)) is
  applied. A board configuration can change it, and *board* means the value
  depends on the board. `defaults` in the CLI resets to the board's own
  defaults.
- Some settings only exist when the firmware includes the feature (LED
  strip, SD card, a built-in SPI receiver, the F.Bus mux FPGA, a particular
  MCU), so `get` on a given board may not list all of them.

Gripora stabilises only the steering (yaw). Roll and pitch keep their
settings in the PID and rate profiles so backups and MSP keep their layout,
but they hold zero and have no effect on a car.

### Gyro, filters and accelerometer

Board orientation, gyro and accelerometer filtering. The three-axis attitude estimate still runs; only the steering loop uses the gyro for control. The RPM filter needs an RPM source on the motor.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `gyro_to_use` | global (hw) | `FIRST`, `SECOND`, `BOTH` | `FIRST` | Which gyro to use on boards with two |
| `gyro_hardware_lpf` | global | `NORMAL`, `OPTION_1`, `OPTION_2`, `EXPERIMENTAL` | `NORMAL` | Gyro chip's own low-pass filter setting |
| `gyro_overflow_detect` | global | `OFF`, `YAW`, `ALL` | `ALL` | Detect gyro overflow on these axes and reset the PID loop |
| `gyro_high_range` | global | `OFF`, `ON` | `OFF` | Use the ICM20649's high measuring range |
| `gyro_rate_sync` | global | `OFF`, `ON` | `ON` | Synchronise the loop to the gyro's sample clock |
| `gyro_calib_duration` | global | `50..3000` | `200` | Gyro calibration time at power-up, 0.01 s |
| `gyro_calib_noise_limit` | global | `0..200` | `100` | Movement allowed during gyro calibration; calibration restarts above it |
| `gyro_decimation_hz` | global | `0..1000` | `500` | Gyro decimation (anti-alias) filter cutoff, Hz; 0 = off |
| `gyro_lpf1_type` | global | `NONE`, `FIRST_ORDER`, `SECOND_ORDER`, `PT1`, `PT2`, `PT3`, `ORDER1`, `BUTTER`, `BESSEL`, `DAMPED` | `FIRST_ORDER` | First gyro low-pass filter type |
| `gyro_lpf1_static_hz` | global | `0..1000` | `100` | First gyro low-pass cutoff, Hz |
| `gyro_lpf1_dyn_min_hz` | global | `0..1000` | `0` | Dynamic low-pass minimum cutoff, Hz; 0 = static filter |
| `gyro_lpf1_dyn_max_hz` | global | `0..1000` | `0` | Dynamic low-pass maximum cutoff, Hz |
| `gyro_lpf2_type` | global | `NONE`, `FIRST_ORDER`, `SECOND_ORDER`, `PT1`, `PT2`, `PT3`, `ORDER1`, `BUTTER`, `BESSEL`, `DAMPED` | `NONE` | Second gyro low-pass filter type |
| `gyro_lpf2_static_hz` | global | `0..1000` | `50` | Second gyro low-pass cutoff, Hz |
| `gyro_notch1_hz` | global | `0..1000` | `0` | Static notch 1 centre, Hz; 0 = off |
| `gyro_notch1_cutoff` | global | `0..1000` | `0` | Static notch 1 lower cutoff, Hz |
| `gyro_notch2_hz` | global | `0..1000` | `0` | Static notch 2 centre, Hz; 0 = off |
| `gyro_notch2_cutoff` | global | `0..1000` | `0` | Static notch 2 lower cutoff, Hz |
| `dyn_notch_count` | global | `0..8` | `6` | Number of dynamic notches; 0 = off |
| `dyn_notch_q` | global | `10..100` | `25` | Dynamic notch Q, x10 |
| `dyn_notch_min_hz` | global | `10..200` | `20` | Lowest dynamic notch frequency, Hz |
| `dyn_notch_max_hz` | global | `100..500` | `240` | Highest dynamic notch frequency, Hz |
| `acc_hardware` | global | `AUTO`, `NONE`, `ADXL345`, `MPU6050`, `MMA8452`, `BMA280`, `LSM303DLHC`, `MPU6000`, `MPU6500`, `MPU9250`, `ICM20601`, `ICM20602`, `ICM20608G`, `ICM20649`, `ICM20689`, `ICM42605`, `ICM42688P`, `BMI160`, `BMI270`, `LSM6DSO`, `BMI088`, `BMI323`, `FAKE` | `AUTO` | Accelerometer chip |
| `acc_high_range` | global | `OFF`, `ON` | `OFF` | Use the ICM20649's high accelerometer range |
| `acc_lpf_hz` | global | `0..400` | `10` | Accelerometer low-pass cutoff, Hz |
| `acc_trim_pitch` | global | `-300..300` | `0` | Accelerometer level trim, pitch |
| `acc_trim_roll` | global | `-300..300` | `0` | Accelerometer level trim, roll |
| `acc_calibration` | global | array of 4 | `0,0,0,0` | Calibration result: three axis offsets and a calibrated flag |
| `align_board_roll` | global | `-180..360` | `0` | Board mounting angle, roll, degrees |
| `align_board_pitch` | global | `-180..360` | `0` | Board mounting angle, pitch, degrees |
| `align_board_yaw` | global | `-180..360` | `0` | Board mounting angle, yaw, degrees |
| `align_board_trim_roll` | global | `-3600..3600` | `0` | Fine trim of the board alignment, 0.1° |
| `align_board_trim_pitch` | global | `-3600..3600` | `0` | Fine trim of the board alignment, 0.1° |
| `align_board_trim_yaw` | global | `-3600..3600` | `0` | Fine trim of the board alignment, 0.1° |
| `imu_dcm_kp` | global | `0..32000` | `2500` | Attitude estimator proportional gain |
| `imu_dcm_ki` | global | `0..32000` | `0` | Attitude estimator integral gain |
| `gyro_1_bustype` | global (hw) | `NONE`, `I2C`, `SPI`, `SLAVE` | board | Gyro 1 bus |
| `gyro_1_spibus` | global (hw) | `0..4` | board | Gyro 1 SPI bus |
| `gyro_1_i2cBus` | global (hw) | 0..I2C bus count | board | Gyro 1 I2C bus |
| `gyro_1_i2c_address` | global (hw) | `0..119` | board | Gyro 1 I2C address |
| `gyro_1_sensor_align` | global (hw) | `DEFAULT`, `CW0`, `CW90`, `CW180`, `CW270`, `CW0FLIP`, `CW90FLIP`, `CW180FLIP`, `CW270FLIP`, `CUSTOM` | board | Gyro 1 orientation |
| `gyro_1_align_roll` | global (hw) | `-3600..3600` | board | Gyro 1 custom alignment, 0.1° |
| `gyro_1_align_pitch` | global (hw) | `-3600..3600` | board | Gyro 1 custom alignment, 0.1° |
| `gyro_1_align_yaw` | global (hw) | `-3600..3600` | board | Gyro 1 custom alignment, 0.1° |
| `gyro_2_bustype` | global (hw) | `NONE`, `I2C`, `SPI`, `SLAVE` | board | Gyro 2 bus |
| `gyro_2_spibus` | global (hw) | `0..4` | board | Gyro 2 SPI bus |
| `gyro_2_i2cBus` | global (hw) | 0..I2C bus count | board | Gyro 2 I2C bus |
| `gyro_2_i2c_address` | global (hw) | `0..119` | board | Gyro 2 I2C address |
| `gyro_2_sensor_align` | global (hw) | `DEFAULT`, `CW0`, `CW90`, `CW180`, `CW270`, `CW0FLIP`, `CW90FLIP`, `CW180FLIP`, `CW270FLIP`, `CUSTOM` | board | Gyro 2 orientation |
| `gyro_2_align_roll` | global (hw) | `-3600..3600` | board | Gyro 2 custom alignment, 0.1° |
| `gyro_2_align_pitch` | global (hw) | `-3600..3600` | board | Gyro 2 custom alignment, 0.1° |
| `gyro_2_align_yaw` | global (hw) | `-3600..3600` | board | Gyro 2 custom alignment, 0.1° |
| `gyro_rpm_notch_preset` | global | `0..3` | `2` | RPM filter preset |
| `gyro_rpm_notch_min_hz` | global | `1..100` | `20` | Lowest RPM notch frequency, Hz |
| `gyro_rpm_notch_source_pitch` | global | array of 16 | preset | Custom RPM filter: RPM source per notch, pitch |
| `gyro_rpm_notch_center_pitch` | global | array of 16 | preset | Custom RPM filter: centre correction per notch, pitch |
| `gyro_rpm_notch_q_pitch` | global | array of 16 | preset | Custom RPM filter: Q per notch (x10), pitch |
| `gyro_rpm_notch_source_roll` | global | array of 16 | preset | Custom RPM filter: RPM source per notch, roll |
| `gyro_rpm_notch_center_roll` | global | array of 16 | preset | Custom RPM filter: centre correction per notch, roll |
| `gyro_rpm_notch_q_roll` | global | array of 16 | preset | Custom RPM filter: Q per notch (x10), roll |
| `gyro_rpm_notch_source_yaw` | global | array of 16 | preset | Custom RPM filter: RPM source per notch, yaw |
| `gyro_rpm_notch_center_yaw` | global | array of 16 | preset | Custom RPM filter: centre correction per notch, yaw |
| `gyro_rpm_notch_q_yaw` | global | array of 16 | preset | Custom RPM filter: Q per notch (x10), yaw |

### Compass and barometer

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `align_mag` | global (hw) | `DEFAULT`, `CW0`, `CW90`, `CW180`, `CW270`, `CW0FLIP`, `CW90FLIP`, `CW180FLIP`, `CW270FLIP`, `CUSTOM` | `DEFAULT` | Magnetometer orientation |
| `mag_align_roll` | global (hw) | `-3600..3600` | `0` | Custom magnetometer alignment, 0.1° |
| `mag_align_pitch` | global (hw) | `-3600..3600` | `0` | Custom magnetometer alignment, 0.1° |
| `mag_align_yaw` | global (hw) | `-3600..3600` | `0` | Custom magnetometer alignment, 0.1° |
| `mag_bustype` | global (hw) | `NONE`, `I2C`, `SPI`, `SLAVE` | board | Magnetometer bus |
| `mag_i2c_device` | global (hw) | `0..4` | board | Magnetometer I2C bus |
| `mag_i2c_address` | global (hw) | `0..119` | board | Magnetometer I2C address |
| `mag_spi_device` | global (hw) | `0..4` | board | Magnetometer SPI bus |
| `mag_hardware` | global | `AUTO`, `NONE`, `HMC5883`, `AK8975`, `AK8963`, `QMC5883`, `LIS3MDL`, `MAG_MPU925X_AK8963` | `NONE` | Magnetometer chip |
| `mag_calibration` | global | array of 3 | `0,0,0` | Magnetometer calibration offsets |
| `baro_bustype` | global (hw) | `NONE`, `I2C`, `SPI`, `SLAVE` | board | Barometer bus |
| `baro_spi_device` | global (hw) | `0..5` | board | Barometer SPI bus |
| `baro_i2c_device` | global (hw) | `0..5` | board | Barometer I2C bus |
| `baro_i2c_address` | global (hw) | `0..119` | board | Barometer I2C address |
| `baro_hardware` | global | `AUTO`, `NONE`, `BMP085`, `MS5611`, `BMP280`, `LPS`, `QMP6988`, `BMP388`, `DPS310`, `BMP581`, `FAKE` | `AUTO` | Barometer chip |

### Receiver

`rc_center` is both the stick centre and the **throttle neutral**: forward above it, brake/reverse below it. The `map` command sets the channel order (default `RT1234AE`: CH1 steering, CH2 throttle, CH3-CH6 AUX1-AUX4). See [Receivers](receivers.md) for `rxfail` and signal loss.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `rx_pulse_min` | global | `750..2250` | `885` | Lowest channel value accepted as valid, µs |
| `rx_pulse_max` | global | `750..2250` | `2115` | Highest channel value accepted as valid, µs |
| `rssi_channel` | global | `0..24` | `0` | Channel carrying RSSI; 0 = none |
| `rssi_src_frame_errors` | global | `OFF`, `ON` | `OFF` | Derive RSSI from the receiver's frame errors |
| `rssi_scale` | global | `1..255` | `100` | RSSI scale |
| `rssi_offset` | global | `-100..100` | `0` | RSSI offset |
| `rssi_invert` | global | `OFF`, `ON` | `OFF` | Invert RSSI |
| `rssi_src_frame_lpf_period` | global | `0..255` | `30` | Filter period for frame-error RSSI, 0.1 s |
| `serialrx_provider` | global | `SPEK1024`, `SPEK2048`, `SBUS`, `SUMD`, `SUMH`, `XB-B`, `XB-B-RJ01`, `IBUS`, `JETIEXBUS`, `CRSF`, `SRXL`, `CUSTOM`, `FPORT`, `SRXL2`, `GHST`, `SBUS2`, `FPORT2`, `FBUS`, `XB-A`, `IBUS2` | `SPEK1024` | Serial receiver protocol |
| `serialrx_inverted` | global | `OFF`, `ON` | `OFF` | Invert the serial receiver signal |
| `serialrx_halfduplex` | global | `OFF`, `ON` | `OFF` | Serial receiver on one wire (half duplex) |
| `serialrx_pinswap` | global | `OFF`, `ON` | `OFF` | Swap the RX and TX pins of the receiver port |
| `spektrum_sat_bind` | global | `0..10` | `0` | Spektrum satellite bind pulses; 0 = off |
| `spektrum_sat_bind_autoreset` | global | `OFF`, `ON` | `ON` | Turn spektrum_sat_bind off again after binding |
| `srxl2_unit_id` | global | `0..15` | `1` | SRXL2 unit id |
| `srxl2_baud_fast` | global | `OFF`, `ON` | `ON` | SRXL2 at 400000 baud |
| `sbus_baud_fast` | global | `OFF`, `ON` | `OFF` | S.Bus at the fast (200000) baud rate |
| `crsf_use_rx_snr` | global | `OFF`, `ON` | `OFF` | Report CRSF SNR instead of RSSI |
| `crsf_use_negotiated_baud` | global | `OFF`, `ON` | `OFF` | Let a CRSF receiver negotiate a faster baud rate |
| `rx_spi_protocol` | global (hw) | `V202_250K`, `V202_1M`, `SYMA_X`, `SYMA_X5C`, `CX10`, `CX10A`, `H8_3D`, `INAV`, `FRSKY_D`, `FRSKY_X`, `FLYSKY`, `FLYSKY_2A`, `KN`, `SFHSS`, `SPEKTRUM`, `FRSKY_X_LBT`, `REDPINE`, `FRSKY_X_V2`, `FRSKY_X_LBT_V2`, `EXPRESSLRS` | board | Protocol of an SPI (on-board) receiver |
| `rx_spi_bus` | global (hw) | 0..SPI bus count | board | SPI bus of the on-board receiver |
| `rx_spi_led_inversion` | global (hw) | `OFF`, `ON` | board | Invert the on-board receiver's LED |
| `input_filtering_mode` | global | `OFF`, `ON` | `OFF` | Hardware filtering of a PWM receiver's inputs |
| `rc_center` | global | `1200..1700` | `1500` | Stick centre and throttle neutral, µs |
| `rc_deflection` | global | `250..750` | `510` | Stick travel from centre to full, µs |
| `rc_min_throttle` | global | `0..2250` | `0` | Stored; car throttle uses rc_center and rc_deflection |
| `rc_max_throttle` | global | `0..2250` | `0` | Stored; car throttle uses rc_center and rc_deflection |
| `rc_smoothness` | global | `0..250` | `50` | RC smoothing level |
| `rc_threshold` | global | array of 3 | `25,25,25` | Stick activity threshold, roll, pitch, yaw |
| `roll_deadband` | global | `0..100` | `5` | Roll stick deadband, µs |
| `pitch_deadband` | global | `0..100` | `5` | Pitch stick deadband, µs |
| `yaw_deadband` | global | `0..100` | `5` | Steering stick deadband, µs |
| `frsky_spi_autobind` | global | `OFF`, `ON` | `OFF` | Built-in FrSky receiver: bind at every start |
| `frsky_spi_tx_id` | global | array of 3 | `0,0,0` | Built-in FrSky receiver: bound transmitter id |
| `frsky_spi_offset` | global | `-127..127` | `0` | Built-in FrSky receiver: frequency offset |
| `frsky_spi_bind_hop_data` | global | array of 50 | all 0 | Built-in FrSky receiver: hop table |
| `frsky_x_rx_num` | global | `0..255` | `0` | Built-in FrSky receiver: receiver number |
| `frsky_spi_a1_source` | global | `VBAT`, `EXTADC`, `CONST` | `VBAT` | Built-in FrSky receiver: A1 telemetry source |
| `cc2500_spi_chip_detect` | global (hw) | `OFF`, `ON` | board | Built-in FrSky receiver: detect the CC2500 chip |
| `spektrum_spi_protocol` | global | `0..255` | `0` | Built-in Spektrum receiver protocol |
| `spektrum_spi_mfg_id` | global | array of 4 | `0,0,0,0` | Built-in Spektrum receiver: bound id |
| `spektrum_spi_num_channels` | global | `0..12` | `0` | Built-in Spektrum receiver: channel count |
| `expresslrs_uid` | global | array of 6 | `0,0,0,0,0,0` | Built-in ELRS receiver: binding UID |
| `expresslrs_domain` | global | the receiver chip's bands | first band | Built-in ELRS receiver: frequency domain |
| `expresslrs_rate_index` | global | `0..3` | `0` | Built-in ELRS receiver: packet rate index |
| `expresslrs_switch_mode` | global | `HYBRID`, `WIDE` | `HYBRID` | Built-in ELRS receiver: switch mode |
| `expresslrs_model_id` | global | `0..255` | `255` | Built-in ELRS receiver: model id |
| `flysky_spi_tx_id` | global | `0..4294967295` | `0` | Built-in FlySky receiver: bound transmitter id |
| `flysky_spi_rf_channels` | global | array of 16 | all 0 | Built-in FlySky receiver: RF channel map |
| `rx_input_backup_provider` | global | `NONE`, `SBUS`, `FBUS`, `FPORT`, `FPORT2`, `EXBUS`, `CRSF` | `NONE` | Backup receiver 1 protocol; NONE = off |
| `rx_input_backup_pinswap` | global | `OFF`, `ON` | `OFF` | Backup receiver 1 pin swap |
| `rx_input_backup_inverted` | global | `OFF`, `ON` | `OFF` | Invert backup receiver 1 |
| `rx_input_backup_halfduplex` | global | `OFF`, `ON` | `OFF` | Backup receiver 1 on one wire |
| `rx_input_backup2_provider` | global | `NONE`, `SBUS`, `FBUS`, `FPORT`, `FPORT2`, `EXBUS`, `CRSF` | `NONE` | Backup receiver 2 protocol; NONE = off |
| `rx_input_backup2_pinswap` | global | `OFF`, `ON` | `OFF` | Backup receiver 2 pin swap |
| `rx_input_backup2_inverted` | global | `OFF`, `ON` | `OFF` | Invert backup receiver 2 |
| `rx_input_backup2_halfduplex` | global | `OFF`, `ON` | `OFF` | Backup receiver 2 on one wire |

### Drive state

How the controller tells the travel direction and speed, and how the steering gyro is faded at low speed and in reverse. See [Drive State](../driving/drive-state.md).

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `drive_esc_mode` | global | `AUTO`, `FWD_BRK_REV`, `FWD_REV`, `FWD_BRK` | `AUTO` | What the ESC does below neutral: AUTO, FWD_BRK_REV (brake, then reverse), FWD_REV (reverse directly), FWD_BRK (never reverses) |
| `drive_wheel_diameter` | global | `20..300` | `100` | Driven tyre diameter, mm, for wheel speed from motor RPM |
| `drive_gyro_min_speed` | global | `10..1999` | `20` | Below this speed the gyro doesn't correct, 0.1 km/h. Needs motor RPM |
| `drive_gyro_full_speed` | global | `1..2000`, kept above `drive_gyro_min_speed` | `60` | Full gyro correction from this speed, 0.1 km/h; always above drive_gyro_min_speed |
| `drive_gyro_reverse` | global | `OFF`, `ON` | `ON` | Gyro correction while reversing |
| `drive_gyro_reverse_gain` | global | `0..100` | `50` | Correction while reversing, % of forward |

### PID profile

Select the PID profile with `profile <n>` first. Only the yaw (steering) gains act; the roll and pitch rows are kept for compatibility. The speed attenuation (`fw_spa_*`) uses the motor RPM wheel speed when there is an RPM source, GPS speed otherwise.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `gain_curve` | PID profile | array of 3 | `0,0,0` | Gain curve (1-8) applied to each axis's master gain by stick deflection, roll, pitch, yaw; 0 = none |
| `profile_name` | PID profile | text | (empty) | PID profile name |
| `pid_mode` | PID profile | `0..9` | `1` | 0 = feedforward only (no gyro), any other value = stabilised rate PID |
| `master_gain` | PID profile | array of 3 | `100,100,100` | P, I and D scale per axis (roll, pitch, yaw), % |
| `fw_tpa_gain` | PID profile | `25..200` | `100` | Throttle attenuation of P, D, F and B: baseline, % |
| `fw_tpa_curve` | PID profile | `0..8` | `0` | Gain curve (1-8) evaluated on throttle magnitude; 0 = off |
| `fw_spa_gain` | PID profile | `25..200` | `100` | Speed attenuation of P, D, F and B: baseline, % |
| `fw_spa_curve` | PID profile | `0..8` | `0` | Gain curve (1-8) evaluated on speed (motor RPM wheel speed, else GPS); 0 = speed attenuation off |
| `fw_spa_speed_max` | PID profile | `10..300` | `60` | Speed at the curve's right edge, km/h |
| `pitch_p_gain` | PID profile | `0..1000` | `105` | Unused on a car (pitch holds zero) |
| `pitch_i_gain` | PID profile | `0..1000` | `45` | Unused on a car |
| `pitch_d_gain` | PID profile | `0..1000` | `0` | Unused on a car |
| `pitch_f_gain` | PID profile | `50..1000` | `65` | Unused on a car |
| `pitch_b_gain` | PID profile | `0..1000` | `35` | Unused on a car |
| `roll_p_gain` | PID profile | `0..1000` | `105` | Unused on a car (roll holds zero) |
| `roll_i_gain` | PID profile | `0..1000` | `45` | Unused on a car |
| `roll_d_gain` | PID profile | `0..1000` | `0` | Unused on a car |
| `roll_f_gain` | PID profile | `50..1000` | `65` | Unused on a car |
| `roll_b_gain` | PID profile | `0..1000` | `35` | Unused on a car |
| `yaw_p_gain` | PID profile | `0..1000` | `190` | Steering P gain |
| `yaw_i_gain` | PID profile | `0..1000` | `45` | Steering I gain |
| `yaw_d_gain` | PID profile | `0..1000` | `0` | Steering D gain |
| `yaw_b_gain` | PID profile | `0..1000` | `35` | Steering B (setpoint boost) gain |
| `yaw_f_gain` | PID profile | `50..1000` | `65` | Steering F (feedforward) gain |
| `pitch_d_cutoff` | PID profile | `0..250` | `15` | Unused on a car |
| `pitch_b_cutoff` | PID profile | `0..250` | `15` | Unused on a car |
| `pitch_gyro_cutoff` | PID profile | `0..250` | `50` | Unused on a car |
| `roll_d_cutoff` | PID profile | `0..250` | `15` | Unused on a car |
| `roll_b_cutoff` | PID profile | `0..250` | `15` | Unused on a car |
| `roll_gyro_cutoff` | PID profile | `0..250` | `50` | Unused on a car |
| `yaw_d_cutoff` | PID profile | `0..250` | `20` | Steering D-term filter cutoff, Hz |
| `yaw_b_cutoff` | PID profile | `0..250` | `20` | Steering B-term filter cutoff, Hz |
| `yaw_gyro_cutoff` | PID profile | `0..250` | `100` | Steering gyro filter cutoff, Hz |
| `error_limit` | PID profile | array of 3 | `45,45,60` | Error limit per axis (roll, pitch, yaw), degrees |
| `iterm_decay_time` | PID profile | array of 3 | `60,60,60` | I-term decay time per axis, 0.01 s |
| `iterm_decay_limit` | PID profile | `0..250` | `35` | — |
| `iterm_relax_level` | PID profile | array of 3 | `22,22,22` | I-term relax level per axis |
| `iterm_relax` | PID profile | array of 3 | `5,5,5` | I-term relax per axis |

### Drive aids (per PID profile)

ABS and traction control, one set per PID profile (select it with `profile <n>` first; a profile switch changes them with the tune). Everything is off by default. They act only while armed, outside failsafe and with the DRIVE AIDS OFF mode off. See [Drive Aids](../driving/drive-aids.md).

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `drive_abs_mode` | PID profile | `OFF`, `STATIC`, `DYNAMIC` | `OFF` | ABS: OFF, STATIC (fixed pulses) or DYNAMIC (pulses follow brake depth) |
| `drive_abs_freq` | PID profile | `2..33` | `10` | ABS pulse rate, Hz |
| `drive_abs_floor` | PID profile | `0..100` | `40` | Brake kept while a pulse releases, % of the brake command (never below 10 % brake) |
| `drive_abs_hold` | PID profile | `0..100` | `20` | Full brake per pulse beyond half the pulse, ms (at most 85 % of the pulse) |
| `drive_abs_trigger` | PID profile | `5..100` | `30` | ABS runs past this brake command, % |
| `drive_abs_dyn_freq` | PID profile | `2..33` | `15` | DYNAMIC: pulse rate at full brake, Hz |
| `drive_abs_dyn_hold` | PID profile | `0..100` | `0` | DYNAMIC: hold at full brake, ms; 0 = off |
| `drive_abs_corner_freq` | PID profile | `0..10` | `0` | Pulse rate added at full steering lock, Hz; 0 = off |
| `drive_abs_corner_hold` | PID profile | `0..100` | `0` | Hold at full steering lock, ms; 0 = off |
| `drive_tc_mode` | PID profile | `OFF`, `RAMP` | `OFF` | Traction control: OFF or RAMP |
| `drive_tc_ramp_time` | PID profile | `50..2000` | `500` | RAMP: quickest zero-to-full forward throttle, ms |

### Rate profile

Select the rate profile with `rateprofile <n>` first. Only the yaw (steering) rows act on a car.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `rateprofile_name` | rate profile | text | (empty) | Rate profile name |
| `roll_rc_rate` | rate profile | `1..200` | `50` | Roll rate (unused on a car) |
| `pitch_rc_rate` | rate profile | `1..200` | `40` | Pitch rate (unused on a car) |
| `yaw_rc_rate` | rate profile | `1..200` | `30` | Full-stick steering rate, x5 °/s (30 = 150 °/s) |
| `roll_expo` | rate profile | `0..100` | `30` | Roll expo (unused on a car) |
| `pitch_expo` | rate profile | `0..100` | `30` | Pitch expo (unused on a car) |
| `yaw_expo` | rate profile | `0..100` | `30` | Steering expo, % |
| `roll_srate` | rate profile | `0..100` | `5` | Roll super rate (unused on a car) |
| `pitch_srate` | rate profile | `0..100` | `5` | Pitch super rate (unused on a car) |
| `yaw_srate` | rate profile | `0..100` | `5` | Shape of the steering expo curve |
| `roll_accel_limit` | rate profile | `0..50000` | `0` | Roll setpoint acceleration limit (unused on a car) |
| `pitch_accel_limit` | rate profile | `0..50000` | `0` | Pitch setpoint acceleration limit (unused on a car) |
| `yaw_accel_limit` | rate profile | `0..50000` | `0` | Steering setpoint acceleration limit; 0 = off |
| `roll_level_expo` | rate profile | `0..100` | `0` | Stored only, no effect |
| `pitch_level_expo` | rate profile | `0..100` | `0` | Stored only, no effect |
| `roll_response` | rate profile | `0..250` | `0` | Roll response time (unused on a car) |
| `pitch_response` | rate profile | `0..250` | `0` | Pitch response time (unused on a car) |
| `yaw_response` | rate profile | `0..250` | `0` | Steering setpoint response time; 0 = off |
| `setpoint_boost_gain` | rate profile | array of 3 | `0,0,0` | Setpoint boost gain, roll, pitch, yaw |
| `setpoint_boost_cutoff` | rate profile | array of 3 | `15,15,90` | Setpoint boost cutoff, roll, pitch, yaw |
| `yaw_dynamic_ceiling_gain` | rate profile | `0..250` | `0` | — |
| `yaw_dynamic_deadband_gain` | rate profile | `0..250` | `10` | — |
| `yaw_dynamic_deadband_filter` | rate profile | `0..250` | `60` | — |
| `yaw_dynamic_deadband_cutoff` | rate profile | `0..250` | `50` | — |

### PID loop and mixer

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `model_type` | global | `CAR`, `CUSTOM` | `CAR` | Mixer model: CAR (car preset) or CUSTOM (rules built by hand) |
| `pid_process_denom` | global | `1..16` | `8` | PID loop rate = gyro rate / this |
| `filter_process_denom` | global | `0..16` | `0` | Filter loop rate divider; 0 = same as the PID loop |
| `yaw_gyro_reverse` | global | `OFF`, `ON` | `OFF` | Reverse only the steering gyro's feedback, for a gyro that corrects the wrong way |

### Motors and ESC

Throttle is signed around neutral. With the defaults, 1000 µs is full brake/reverse, 1500 µs neutral and 2000 µs full forward, and every motor is bidirectional.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `freq_input_pull` | global | `NOPULL`, `PULLUP`, `PULLDOWN` | `PULLUP` | Pull resistor on the frequency (speed sensor) input |
| `freq_input_edge` | global | `FALLING`, `RISING` | `FALLING` | Edge counted on the frequency input |
| `freq_input_minhz` | global | `1..100` | `25` | Lowest frequency the input measures, Hz |
| `min_throttle` | global | `50..2250` | `1000` | ESC pulse at full brake/reverse, µs. Neutral is the midpoint of min_throttle and max_throttle |
| `max_throttle` | global | `50..2250` | `2000` | ESC pulse at full forward throttle, µs |
| `min_command` | global | `50..2250` | `1000` | Pulse sent to a motor that is not bidirectional when stopped, µs |
| `dshot_burst` | global (hw) | `OFF`, `ON`, `AUTO` | `OFF` | DShot DMA burst mode |
| `dshot_bidir` | global | `OFF`, `ON` | `OFF` | Bidirectional DShot (RPM telemetry) |
| `dshot_bitbang` | global (hw) | `OFF`, `ON`, `AUTO` | `AUTO` | DShot bit-banging |
| `dshot_bitbang_timer` | global (hw) | `AUTO`, `TIM1`, `TIM8` | `AUTO` | Timer used for DShot bit-banging |
| `use_unsynced_pwm` | global | `OFF`, `ON` | `OFF` | Run motor PWM at its own rate instead of the loop rate. Forced on for PWM and Castle |
| `motor_pwm_protocol` | global | `PWM`, `ONESHOT125`, `ONESHOT42`, `MULTISHOT`, `RESERVED`, `DSHOT150`, `DSHOT300`, `DSHOT600`, `PROSHOT1000`, `CASTLE`, `SRXL2`, `DISABLED` | `PWM` | ESC protocol |
| `motor_pwm_rate` | global | `50..8000` | `50` | Motor PWM rate, Hz |
| `motor_control_mode` | global | array of 4 | `1,1,1,1` | Per motor: 0 = one direction, 1 = bidirectional (forward and brake/reverse around neutral) |
| `motor_poles` | global | array of 4 | `0,0,0,0` | Motor pole count, for RPM from eRPM |
| `motor_rpm_lpf` | global | array of 4 | `100,100,100,100` | Motor RPM low-pass cutoff, Hz |
| `motor_rpm_factor` | global | array of 4 | `0,0,0,0` | Motor RPM correction factor |
| `motor1_gear_ratio` | global | array of 2 | `1,1` | Motor 1 final drive as two numbers: output (wheel) turns, then motor turns. A 10.4 : 1 final drive is `10,104` (the Configurator shows it as motor : wheel). Used for the wheel speed and the RPM filter |
| `motor2_gear_ratio` | global | array of 2 | `1,1` | The same for motor 2 (its speed is averaged with motor 1's when both have RPM) |
| `esc_sensor_protocol` | global | `OFF`, `BLHELI32`, `HOBBYWINGV4`, `HOBBYWINGV5`, `SCORPION`, `KONTRONIK`, `OMPHOBBY`, `ZTW`, `APD`, `OPENYGE`, `FLYROTOR`, `GRAUPNER`, `XDFLY`, `FBUS`, `SRXL2`, `RECORD` | `OFF` | ESC telemetry protocol |
| `esc_sensor_halfduplex` | global | `OFF`, `ON` | `OFF` | ESC telemetry on one wire |
| `esc_sensor_pinswap` | global | `OFF`, `ON` | `OFF` | Swap the RX and TX pins of the ESC telemetry port |
| `esc_sensor_update_hz` | global | `4..500` | `50` | ESC telemetry update rate, Hz |
| `esc_sensor_current_offset` | global | `0..16000` | `0` | ESC current offset |
| `esc_sensor_filter_cutoff` | global | `0..250` | `0` | ESC telemetry filter cutoff; 0 = off |
| `esc_sensor_voltage_correction` | global | `-100..125` | `0` | ESC voltage correction, % |
| `esc_sensor_current_correction` | global | `-100..125` | `0` | ESC current correction, % |
| `esc_sensor_consumption_correction` | global | `-100..125` | `0` | ESC consumption correction, % |
| `srxl2esc_motor2_channel` | global | `0..32` | `0` | SRXL2 ESC: channel for the second motor; 0 = none |

### Servos and bus outputs

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `servo_pwm_rate` | global | `25..333` | `50` | Update rate of every PWM servo output, Hz; applied at boot. 50 suits any servo, higher is for digital servos only |
| `sbus_out_frame_rate` | global | `25..250` | `50` | S.Bus output frame rate, Hz |
| `sbus_out_pinswap` | global | `OFF`, `ON` | `OFF` | S.Bus output pin swap |
| `sbus_out_inverted` | global | `OFF`, `ON` | `ON` | Invert the S.Bus output |
| `sbus_out_channels` | global | `8`, `12`, `16` | `16` | Channels S.Bus output sends (always the 16-channel frame) |
| `fbus_master_frame_rate` | global | `25..550` | `500` | F.Bus output frame rate, Hz |
| `fbus_master_pinswap` | global | `OFF`, `ON` | `OFF` | F.Bus output pin swap |
| `fbus_master_inverted` | global | `OFF`, `ON` | `ON` | Invert the F.Bus output |
| `fbus_master_telemetry_rate` | global | `25..550` | `200` | F.Bus sensor polling rate, Hz |
| `fbus_master_discovery_ms` | global | `100..10000` | `5000` | F.Bus sensor discovery interval, ms |
| `fbus_master_forwarded_sensors` | global | array of 8 | `255,255,255,255,255,255,255,255` | F.Bus sensors forwarded to telemetry; 255 = none |
| `fbus_master_baud_fast` | global | `OFF`, `ON` | `ON` | F.Bus output at 460800 baud (OFF: 115200, frame rate capped at 150 Hz) |
| `fbus_master_channels` | global | `8`, `12`, `16`, `24` | `24` | Channels F.Bus output sends. 8 uses the 8-channel frame, 12 and 16 the 16-channel frame, 24 the 24-channel frame |
| `fpga_spi_bus` | global (hw) | 0..SPI bus count | `0` | F.Bus mux FPGA SPI bus |
| `fpga_serial` | global (hw) | 0..serial port count | `0` | F.Bus mux FPGA serial port |
| `fbus_mux_ch1_mode` | global | `PWM`, `FBUS`, `DSHOT` | `PWM` | F.Bus mux output 1 mode |
| `fbus_mux_ch2_mode` | global | `PWM`, `FBUS`, `DSHOT` | `PWM` | F.Bus mux output 2 mode |
| `fbus_mux_ch3_mode` | global | `PWM`, `FBUS`, `DSHOT` | `PWM` | F.Bus mux output 3 mode |
| `fbus_mux_ch4_mode` | global | `PWM`, `FBUS`, `DSHOT` | `PWM` | F.Bus mux output 4 mode |
| `fbus_mux_ch5_mode` | global | `PWM`, `FBUS`, `DSHOT` | `PWM` | F.Bus mux output 5 mode |
| `sport_master_pinswap` | global | `OFF`, `ON` | `ON` | S.Port master pin swap |
| `sport_master_inverted` | global | `OFF`, `ON` | — | Invert the S.Port master signal |
| `crsf_sensors_timeout_ms` | global | `500..10000` | `2000` | CRSF sensor input: timeout, ms |
| `crsf_sensors_use_baro` | global | `OFF`, `ON` | `OFF` | CRSF sensor input: use its barometer |
| `crsf_sensors_pinswap` | global | `OFF`, `ON` | `ON` | CRSF sensor input: pin swap |
| `crsf_sensors_battery_source` | global | `AUTO`, `CURRENT`, `VOLTAGE` | `AUTO` | CRSF sensor input: battery source |
| `crsf_sensors_use_rpm` | global | `OFF`, `ON` | `OFF` | CRSF sensor input: use its RPM |

### Arming and failsafe

Arming needs the throttle at neutral. See [Arming and Stick Commands](stick-commands.md) and the [Failsafe tab](../configurator/tabs/failsafe.md).

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `failsafe_delay` | global | `2..200` | `15` | Stage 1 length before stage 2 starts, 0.1 s |
| `failsafe_off_delay` | global | `0..200` | `10` | AUTO-LAND: how long failsafe_throttle is held before disarming, 0.1 s |
| `failsafe_throttle` | global | `750..2250` | `1500` | Throttle held during AUTO-LAND, µs; 1500 is neutral |
| `failsafe_switch_mode` | global | `STAGE1`, `KILL`, `STAGE2` | `STAGE1` | What the FAILSAFE mode switch does: stage 1, kill (disarm at once) or stage 2 |
| `failsafe_throttle_low_delay` | global | `0..300` | `100` | Disarm at once on signal loss if the throttle was at neutral this long, 0.1 s; 0 = off |
| `failsafe_procedure` | global | `AUTO-LAND`, `DROP` | `DROP` | Stage 2: DROP disarms, AUTO-LAND holds failsafe_throttle for failsafe_off_delay, then disarms |
| `failsafe_recovery_delay` | global | `0..200` | `10` | Valid signal needed to leave failsafe, 0.1 s |
| `auto_disarm_delay` | global | `0..60` | `5` | Stick arming only: disarm after this long at neutral throttle, s; 0 = off |
| `gyro_cal_on_first_arm` | global | `OFF`, `ON` | `OFF` | Recalibrate the gyro on the first arm |
| `pwr_on_arm_grace` | global | `0..30` | `3` | No arming for this long after power-up, s |
| `rearm_grace_seconds` | global | `0..30` | `10` | After a disarm while the car was running, re-arm within this time without the throttle and attitude checks, s; 0 = off |
| `rearm_min_armed_seconds` | global | `0..30` | `5` | The car must have been armed this long for rearm_grace_seconds to apply, s |
| `enable_stick_arming` | global | `OFF`, `ON` | `OFF` | Arm and disarm with stick positions (no ARM switch set) |
| `enable_stick_commands` | global | `OFF`, `ON` | `OFF` | Stick commands while disarmed (profiles, calibration, save) |
| `wiggle_strength` | global | `0..100` | `50` | Servo wiggle size, % |
| `wiggle_frequency` | global | `2..50` | `10` | Servo wiggle rate, Hz |
| `wiggle_enable_ready` | global | `OFF`, `ON` | `ON` | Wiggle the steering when ready to arm |
| `wiggle_enable_armed` | global | `OFF`, `ON` | `OFF` | Wiggle the steering on arming |
| `wiggle_enable_error` | global | `OFF`, `ON` | `OFF` | Wiggle an error code when arming is refused |
| `wiggle_enable_fatal` | global | `OFF`, `ON` | `OFF` | Wiggle on a fatal error |

### Battery and power

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `adc_device` | global (hw) | 0..ADC count | board | ADC used for voltage and current inputs |
| `adc_vrefint_calibration` | global | `0..2000` | board | ADC reference calibration |
| `adc_tempsensor_calibration30` | global | `0..2000` | board | MCU temperature sensor calibration at 30 °C |
| `adc_tempsensor_calibration110` | global | `0..2000` | board | MCU temperature sensor calibration at 110 °C |
| `bat_profile` | global | `0..5` | `0` | Battery profile in use (0-5) |
| `bat_capacity` | global | array of 6 | `0,0,0,0,0,0` | Battery capacity per battery profile, mAh |
| `vbat_max_cell_voltage` | global | array of 6 | `430,430,430,430,430,430` | Maximum cell voltage per battery profile, 0.01 V |
| `vbat_full_cell_voltage` | global | array of 6 | `410,410,410,410,410,410` | Full cell voltage per battery profile, 0.01 V |
| `vbat_min_cell_voltage` | global | array of 6 | `330,330,330,330,330,330` | Minimum (empty) cell voltage per battery profile, 0.01 V |
| `vbat_warning_cell_voltage` | global | array of 6 | `350,350,350,350,350,350` | Low-battery warning cell voltage per battery profile, 0.01 V |
| `vbat_hysteresis` | global | `0..250` | `1` | Voltage hysteresis for the battery warnings, 0.01 V |
| `current_meter` | global (hw) | `NONE`, `ADC`, `ESC`, `FBUS`, `CRSF` | `ESC` | Current source |
| `battery_meter` | global (hw) | `NONE`, `ADC`, `ESC`, `FBUS`, `CRSF` | `ESC` | Voltage source |
| `vbat_detect_cell_voltage` | global | `0..2000` | `300` | Cell voltage used to detect the cell count, 0.01 V |
| `use_vbat_alerts` | global | `OFF`, `ON` | `ON` | Beep on low voltage |
| `use_cbat_alerts` | global | `OFF`, `ON` | `OFF` | Beep on low remaining capacity |
| `cbat_alert_percent` | global | `0..100` | `35` | Remaining capacity alert, % |
| `vbat_cutoff_percent` | global | `0..100` | `100` | — |
| `battery_cell_count` | global | array of 6 | `0,0,0,0,0,0` | Cell count per battery profile; 0 = detect |
| `vbat_lpf_hz` | global | `0..255` | `10` | Voltage filter cutoff, Hz |
| `ibat_lpf_hz` | global | `0..255` | `10` | Current filter cutoff, Hz |
| `vbat_duration_for_warning` | global | `0..150` | `0` | — |
| `vbat_duration_for_critical` | global | `0..150` | `0` | — |
| `vbat_update_hz` | global | `10..1000` | `50` | Voltage update rate, Hz |
| `ibat_update_hz` | global | `10..1000` | `50` | Current update rate, Hz |
| `smartfuel` | global | `OFF`, `VOLTAGE`, `CURRENT`, `COMBINED` | `CURRENT` | SmartFuel remaining-charge estimate source |
| `smartfuel_voltage_drop_rate` | global | `0..250` | `10` | — |
| `smartfuel_charge_drop_rate` | global | `0..250` | `50` | — |
| `smartfuel_sag_gain` | global | `0..100` | `40` | — |
| `vbat_scale` | global (hw) | `0..65535` | `110` | Battery voltage ADC scale |
| `vbat_divider` | global (hw) | `1..65535` | `10` | Battery voltage ADC divider |
| `vbat_multiplier` | global (hw) | `1..255` | `1` | Battery voltage ADC multiplier |
| `vbat_cutoff` | global (hw) | `0..250` | `25` | Battery voltage filter cutoff |
| `vbec_scale` | global (hw) | `0..65535` | `110` | BEC voltage ADC scale |
| `vbec_divider` | global (hw) | `1..65535` | `10` | BEC voltage ADC divider |
| `vbec_multiplier` | global (hw) | `1..255` | `1` | BEC voltage ADC multiplier |
| `vbec_cutoff` | global (hw) | `0..250` | `25` | BEC voltage filter cutoff |
| `vbus_scale` | global (hw) | `0..65535` | `110` | Bus voltage ADC scale |
| `vbus_divider` | global (hw) | `1..65535` | `10` | Bus voltage ADC divider |
| `vbus_multiplier` | global (hw) | `1..255` | `1` | Bus voltage ADC multiplier |
| `vbus_cutoff` | global (hw) | `0..250` | `25` | Bus voltage filter cutoff |
| `vext_scale` | global (hw) | `0..65535` | `110` | External voltage ADC scale |
| `vext_divider` | global (hw) | `1..65535` | `10` | External voltage ADC divider |
| `vext_multiplier` | global (hw) | `1..255` | `1` | External voltage ADC multiplier |
| `vext_cutoff` | global (hw) | `0..250` | `25` | External voltage filter cutoff |
| `ibata_scale` | global (hw) | `-16000..16000` | `400` | Current sensor ADC scale |
| `ibata_offset` | global (hw) | `-32000..32000` | `0` | Current sensor ADC offset |
| `ibata_cutoff` | global (hw) | `0..250` | `25` | Current sensor filter cutoff |

### GPS and position

GPS is optional on a car: it gives the speed for the speed attenuation when the motor has no RPM source, and position and speed for telemetry.

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `gps_provider` | global | `NMEA`, `UBLOX`, `MSP`, `FBUS`, `CRSF` | `NMEA` | GPS protocol |
| `gps_sbas_mode` | global | `AUTO`, `EGNOS`, `WAAS`, `MSAS`, `GAGAN`, `NONE` | `NONE` | SBAS region |
| `gps_sbas_integrity` | global | `OFF`, `ON` | `OFF` | Use only SBAS-corrected fixes |
| `gps_auto_config` | global | `OFF`, `ON` | `ON` | Configure the GPS module at start-up |
| `gps_auto_baud` | global | `OFF`, `ON` | `OFF` | Find the GPS baud rate |
| `gps_ublox_use_galileo` | global | `OFF`, `ON` | `OFF` | Use Galileo satellites (u-blox) |
| `gps_ublox_mode` | global | `AIRBORNE`, `PEDESTRIAN`, `DYNAMIC` | `AIRBORNE` | u-blox dynamic model |
| `gps_set_home_point_once` | global | `OFF`, `ON` | `OFF` | Set the home point only on the first arm |
| `gps_use_3d_speed` | global | `OFF`, `ON` | `OFF` | Use 3D speed instead of ground speed |
| `gps_fbus_assumed_sats` | global | `0..50` | `0` | Satellite count assumed for an F.Bus GPS that doesn't report one |
| `position_alt_source` | global | `DEFAULT`, `BARO_ONLY`, `GPS_ONLY` | `DEFAULT` | Altitude source |
| `position_baro_alt_lpf` | global | `1..250` | `100` | Barometer altitude filter |
| `position_baro_offset_lpf` | global | `1..250` | `5` | Barometer offset filter |
| `position_gps_alt_lpf` | global | `1..250` | `25` | GPS altitude filter |
| `position_gps_offset_lpf` | global | `1..250` | `5` | GPS offset filter |
| `position_gps_min_sats` | global | `0..50` | `6` | Satellites needed before GPS altitude is used |
| `position_vario_lpf` | global | `1..250` | `50` | Vario filter |
| `position_fusion_baro_tc` | global | `5..100` | `20` | How long the accelerometer is trusted over the barometer, 0.1 s |
| `position_fusion_gps_tc` | global | `5..200` | `40` | How long the accelerometer is trusted over GPS altitude, 0.1 s |

### Telemetry

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `tlm_inverted` | global | `OFF`, `ON` | `OFF` | Invert the telemetry signal |
| `tlm_halfduplex` | global | `OFF`, `ON` | `ON` | Telemetry on one wire |
| `tlm_pinswap` | global | `OFF`, `ON` | `OFF` | Swap the RX and TX pins of the telemetry port |
| `frsky_default_lat` | global | `-9000..9000` | `0` | FrSky hub: latitude sent without a GPS fix |
| `frsky_default_long` | global | `-18000..18000` | `0` | FrSky hub: longitude sent without a GPS fix |
| `frsky_gps_format` | global | `0..1` | `0` | FrSky hub: coordinate format |
| `frsky_unit` | global | `IMPERIAL`, `METRIC`, `BRITISH` | `METRIC` | FrSky hub: units |
| `frsky_vfas_precision` | global | `0..1` | `0` | FrSky hub: VFAS precision |
| `hott_alarm_int` | global | `0..120` | `5` | HoTT alarm interval, s |
| `report_cell_voltage` | global | `OFF`, `ON` | `OFF` | Report average cell voltage instead of pack voltage |
| `ibus_sensor` | global | array of 15 | — | IBUS telemetry sensors |
| `mavlink_mah_as_heading_divisor` | global | `0..30000` | `0` | MAVLink: send mAh drawn in the heading field, divided by this; 0 = off |
| `crsf_telemetry_mode` | global | `NATIVE`, `CUSTOM` | `CUSTOM` | CRSF telemetry: NATIVE frames or CUSTOM (read by the Gripora radio scripts) |
| `crsf_telemetry_link_rate` | global | `0..50000` | `250` | CRSF custom telemetry rate |
| `crsf_telemetry_link_ratio` | global | `0..50000` | `8` | CRSF custom telemetry ratio |
| `telemetry_sensors` | global | array of 40 | `3,4,5,6,15,43,50,52,58,59,60,89,91,99,120,121`, the rest 0 (see [Telemetry](telemetry.md)) | Telemetry sensor ids sent (custom CRSF, FBUS/S.Port) |
| `telemetry_interval` | global | array of 40 | all 0 | Send interval per sensor slot |

### Blackbox and debug

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `blackbox_mode` | global | `OFF`, `NORMAL`, `ARMED`, `SWITCH` | `ARMED` | When Blackbox records: always, while armed, or on the BLACKBOX switch |
| `blackbox_device` | global | `NONE`, `SPIFLASH`, `SDCARD`, `SERIAL` | board (flash, SD card or serial) | Where Blackbox logs go |
| `blackbox_rate_denom` | global | `1..8000` | `8` | Log every Nth loop |
| `blackbox_log_command` | global | `OFF`, `ON` | `ON` | Log the RC commands |
| `blackbox_log_setpoint` | global | `OFF`, `ON` | `ON` | Log the setpoints |
| `blackbox_log_mixer` | global | `OFF`, `ON` | `ON` | Log the mixer inputs |
| `blackbox_log_pid` | global | `OFF`, `ON` | `ON` | Log the PID terms |
| `blackbox_log_attitude` | global | `OFF`, `ON` | `ON` | Log the attitude |
| `blackbox_log_gyro_raw` | global | `OFF`, `ON` | `ON` | Log the unfiltered gyro |
| `blackbox_log_gyro` | global | `OFF`, `ON` | `ON` | Log the filtered gyro |
| `blackbox_log_acc` | global | `OFF`, `ON` | `OFF` | Log the accelerometer |
| `blackbox_log_mag` | global | `OFF`, `ON` | `OFF` | Log the magnetometer |
| `blackbox_log_alt` | global | `OFF`, `ON` | `ON` | Log altitude and vario |
| `blackbox_log_gps` | global | `OFF`, `ON` | `OFF` | Log GPS data |
| `blackbox_log_battery` | global | `OFF`, `ON` | `ON` | Log battery voltage and current |
| `blackbox_log_motors` | global | `OFF`, `ON` | `ON` | Log motor outputs |
| `blackbox_log_servos` | global | `OFF`, `ON` | `ON` | Log servo outputs |
| `blackbox_log_rpm` | global | `OFF`, `ON` | `ON` | Log motor RPM |
| `blackbox_log_rssi` | global | `OFF`, `ON` | `ON` | Log RSSI |
| `blackbox_log_vbec` | global | `OFF`, `ON` | `ON` | Log the BEC voltage |
| `blackbox_log_vbus` | global | `OFF`, `ON` | `ON` | Log the bus voltage |
| `blackbox_log_temp` | global | `OFF`, `ON` | `ON` | Log temperatures |
| `blackbox_log_esc` | global | `OFF`, `ON` | `OFF` | Log ESC telemetry |
| `blackbox_log_bec` | global | `OFF`, `ON` | `OFF` | Log BEC telemetry |
| `blackbox_log_esc2` | global | `OFF`, `ON` | `OFF` | Log the second ESC's telemetry |
| `blackbox_initial_erase_kb` | global | `0..65535` | `0` | Flash erased ahead of logging at arming, KB; 0 = off |
| `blackbox_rolling_erase` | global | `OFF`, `ON` | `ON` | Erase flash ahead of the log while logging |
| `blackbox_grace_period` | global | `0..60` | `5` | Keep logging this long after disarm, s |
| `task_statistics` | global | `OFF`, `ON` | `ON` | Collect task timing statistics |
| `debug_mode` | global | see [Debug modes](#debug-modes) | `NONE` | What the Blackbox debug fields record; see Debug modes below |
| `debug_axis` | global | `0..255` | `0` | Axis for the per-axis debug modes |

### Model, statistics and modes

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `stats_min_armed_time_s` | global | `-1..127` | `15` | Shortest armed time counted as a run, s; -1 = statistics off |
| `stats_total_flights` | global | `0..4294967295` | `0` | Run count (the name is inherited from the controller) |
| `stats_total_time_s` | global | `0..4294967295` | `0` | Total armed time, s |
| `stats_total_dist_m` | global | `0..4294967295` | `0` | Total distance, m |
| `name` | global | text | (empty) | Model name |
| `model_id` | global | `0..99` | `0` | Model id the radio scripts match |
| `model_param1_type` | global | `NONE`, `TIMER1`, `TIMER2`, `TIMER3`, `GV1`, `GV2`, `GV3`, `GV4`, `GV5`, `GV6`, `GV7`, `GV8`, `GV9` | `NONE` | Radio parameter 1 type |
| `model_param1_value` | global | `-32768..32767` | `0` | Radio parameter 1 value |
| `model_param2_type` | global | `NONE`, `TIMER1`, `TIMER2`, `TIMER3`, `GV1`, `GV2`, `GV3`, `GV4`, `GV5`, `GV6`, `GV7`, `GV8`, `GV9` | `NONE` | Radio parameter 2 type |
| `model_param2_value` | global | `-32768..32767` | `0` | Radio parameter 2 value |
| `model_param3_type` | global | `NONE`, `TIMER1`, `TIMER2`, `TIMER3`, `GV1`, `GV2`, `GV3`, `GV4`, `GV5`, `GV6`, `GV7`, `GV8`, `GV9` | `NONE` | Radio parameter 3 type |
| `model_param3_value` | global | `-32768..32767` | `0` | Radio parameter 3 value |
| `model_set_name` | global | `OFF`, `ON` | `OFF` | — |
| `model_tell_capacity` | global | `OFF`, `ON` | `OFF` | — |
| `box_user_1_name` | global | text | (empty) | Name shown for USER1 |
| `box_user_2_name` | global | text | (empty) | Name shown for USER2 |
| `box_user_3_name` | global | text | (empty) | Name shown for USER3 |
| `box_user_4_name` | global | text | (empty) | Name shown for USER4 |

### LED strip, buzzer and display

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `beeper_inversion` | global (hw) | `OFF`, `ON` | board | Invert the buzzer output |
| `beeper_od` | global (hw) | `OFF`, `ON` | board | Buzzer output open drain |
| `beeper_frequency` | global (hw) | `0..16000` | board | Buzzer PWM frequency, Hz; 0 = plain on/off buzzer |
| `beeper_dshot_beacon_tone` | global | `1..5` | `1` | Tone of the DShot motor beacon |
| `ledstrip_visual_beeper` | global | `OFF`, `ON` | `OFF` | Flash the LED strip with the beeper |
| `ledstrip_visual_beeper_color` | global | LED colour name | `WHITE` | Colour of the visual beeper |
| `ledstrip_grb_rgb` | global | `GRB`, `RGB`, `GRBW` | `GRB` | LED colour order |
| `ledstrip_profile` | global | `RACE`, `BEACON`, `STATUS`, `STATUS_ALT` | `STATUS` | LED strip profile |
| `ledstrip_race_color` | global | LED colour name | `ORANGE` | Colour in the RACE profile |
| `ledstrip_beacon_color` | global | LED colour name | `WHITE` | Colour in the BEACON profile |
| `ledstrip_beacon_period_ms` | global | `50..10000` | `500` | Beacon period, ms |
| `ledstrip_beacon_percent` | global | `0..100` | `50` | Beacon duty cycle, % |
| `ledstrip_beacon_armed_only` | global | `OFF`, `ON` | `OFF` | Beacon only while armed |
| `ledstrip_brightness` | global | `5..100` | `100` | Brightness, % |
| `ledstrip_blink_period_ms` | global | `10..500` | `100` | Blink period, ms |
| `ledstrip_flicker_rate` | global | `0..100` | `50` | Flicker rate |
| `ledstrip_fade_rate` | global | `0..100` | `50` | Fade rate |
| `ledstrip_inverted_format` | global | `0..4294967295` | `0` | Bit mask of LEDs with inverted colour format |
| `led_inversion` | global (hw) | `0..7` | board | Invert the status LEDs (bit mask) |
| `dashboard_i2c_bus` | global (hw) | 0..I2C bus count | board | OLED display I2C bus |
| `dashboard_i2c_addr` | global (hw) | `8..119` | `60` | OLED display I2C address |
| `pinio_config` | global (hw) | array of 4 | board | PINIO output modes |
| `pinio_box` | global (hw) | array of 4 | board | Mode (permanent id) driving each PINIO; 255 = none |
| `rcdevice_init_dev_attempts` | global | `0..10` | `6` | RunCam device: connection attempts |
| `rcdevice_init_dev_attempt_interval` | global | `0..5000` | `1000` | RunCam device: time between attempts, ms |
| `rcdevice_protocol_version` | global | `0..1` | `0` | RunCam device protocol version |
| `rcdevice_feature` | global | `0..65535` | `0` | RunCam device feature mask |

### System and hardware

| Setting | Scope | Values | Default | Description |
|---|---|---|---|---|
| `reboot_character` | global | `48..126` | `82` | Character that reboots the board into the bootloader over a serial port |
| `serial_update_rate_hz` | global | `100..2000` | `100` | Serial task rate, Hz |
| `sdcard_detect_inverted` | global (hw) | `OFF`, `ON` | board | Invert the SD card detect pin |
| `sdcard_mode` | global (hw) | `OFF`, `SPI`, `SDIO` | board | SD card interface |
| `sdcard_spi_bus` | global (hw) | 0..SPI bus count | board | SD card SPI bus |
| `sdio_clk_bypass` | global (hw) | `OFF`, `ON` | `OFF` | SDIO clock bypass |
| `sdio_use_cache` | global (hw) | `OFF`, `ON` | `OFF` | SDIO cache |
| `sdio_use_4bit_width` | global (hw) | `OFF`, `ON` | board | SDIO 4-bit bus |
| `sdio_device` | global (hw) | 0..SDIO count | board | SDIO device |
| `system_hse_mhz` | global (hw) | `0..30` | board | External crystal frequency, MHz |
| `cpu_overclock` | global | `OFF` and the speeds the MCU supports | `OFF` (`108MHZ` on F411) | CPU overclock |
| `usb_hid_cdc` | global | `OFF`, `ON` | `OFF` | USB HID (joystick) together with the virtual COM port |
| `usb_msc_pin_pullup` | global (hw) | `OFF`, `ON` | `ON` | Pull-up on the USB mass-storage button pin |
| `flash_spi_bus` | global (hw) | 0..SPI bus count | board | SPI bus of the flash chip |
| `i2c1_pullup` | global (hw) | `OFF`, `ON` | board | I2C1 internal pull-ups |
| `i2c1_clockspeed_khz` | global (hw) | `100..1300` | board | I2C1 clock, kHz |
| `i2c2_pullup` | global (hw) | `OFF`, `ON` | board | I2C2 internal pull-ups |
| `i2c2_clockspeed_khz` | global (hw) | `100..1300` | board | I2C2 clock, kHz |
| `i2c3_pullup` | global (hw) | `OFF`, `ON` | board | I2C3 internal pull-ups |
| `i2c3_clockspeed_khz` | global (hw) | `100..1300` | board | I2C3 clock, kHz |
| `i2c4_pullup` | global (hw) | `OFF`, `ON` | board | I2C4 internal pull-ups |
| `i2c4_clockspeed_khz` | global (hw) | `100..1300` | board | I2C4 clock, kHz |
| `mco_on_pa8` | global (hw) | `OFF`, `ON` | board | Clock output on PA8 (G4) |
| `mco_source` | global (hw) | `—` | board | Clock output source |
| `mco_divider` | global (hw) | `—` | board | Clock output divider |
| `mco2_on_pc9` | global (hw) | `OFF`, `ON` | board | Clock output 2 on PC9 |
| `scheduler_relax_rx` | global | `0..500` | `25` | — |
| `timezone_offset_minutes` | global | `-780..780` | `0` | Time zone offset, minutes |

## Removed settings

These WingFlight settings went with the aircraft features (angle and
leveling, acro trainer, attitude hold, GPS rescue and navigation, thrust
vectoring, the governor, cross-axis, snap and prop-hang relax):

- `governor_*`, `gps_rescue_*`, `nav_*`, `tv_*`, `angle_*`,
  `acro_trainer_*`, `atthold_*`, `cross_axis_relax_*`, `snap_relax_*`,
  `prop_hang_*` and `failsafe_stick_threshold`
- the `tv_profile` command

Pasting a `diff` from WingFlight or older Gripora reports these lines as
unknown; they can be ignored. Settings whose range or defaults changed for a
car (for example `model_type`, `map`, `fw_spa_speed_max`, the throttle and
failsafe defaults) were reset to the new defaults when the firmware was
upgraded, so check them after restoring an old backup.

## Debug modes

`debug_mode` picks what the Blackbox debug fields record. Gripora's own
modes:

`DRIVE` shows the [drive state](../driving/drive-state.md):

| Field | Value |
|---|---|
| 0 | Throttle command, x1000 |
| 1 | Commanded ESC state: 0 neutral, 1 forward, 2 brake, 3 reverse |
| 2 | Travel direction (+1 forward, -1 reverse) |
| 3 | Speed, 0.1 km/h |
| 4 | Speed known (1 = there is an RPM source) |
| 5 | Gyro correction scale, x1000 |
| 6 | Motor 1 RPM |
| 7 | Stopped (1 = RPM shows the car stopped) |

`DRIVE_AIDS` shows the [drive aids](../driving/drive-aids.md):

| Field | Value |
|---|---|
| 0 | Throttle from the transmitter, x1000 (below 0 is brake/reverse) |
| 1 | Throttle to the ESC, x1000 |
| 2 | Flags: bit 0 aids on, bit 1 ABS armed, bit 2 ABS releasing, bit 3 traction control armed, bit 4 traction control holding the throttle back |
| 3 | While braking with ABS: pulse rate, x10 Hz. While driving forward with traction control: the throttle it allows, x1000 |
| 4 | ABS release brake (floor), x1000 |
| 5 | ABS hold, ms |
| 6 | Time into the current ABS pulse, ms |
| 7 | Steering, x1000 |

`GAIN_ATTEN` shows the throttle (TPA) and speed (SPA) gain attenuation that
scales P, D, F and B:

| Field | Value |
|---|---|
| 0 | Throttle attenuation, x1000 (1000 = 100%) |
| 1 | Speed attenuation, x1000 |
| 2 | Speed used by the curve, 0.1 km/h (motor RPM wheel speed, else GPS) |
| 3 | Raw GPS speed, cm/s |
| 4 | GPS fix (1 = fix) |

`ALTITUDE` shows the altitude estimate next to its sources:

| Field | Value |
|---|---|
| 0 | Altitude, cm (what telemetry uses) |
| 1 | Vario, cm/s |
| 2 | Measured altitude before fusion, cm |
| 3 | Measured vario (derivative of the measured altitude), cm/s |
| 4 | Barometer altitude, cm |
| 5 | GPS altitude, cm |
| 6 | Vertical acceleration, gravity removed, cm/s² |
| 7 | Learned accelerometer error, cm/s² |

Altitude comes from the accelerometer fused with a measured altitude: the
barometer, or GPS altitude on boards without one (`position_*` settings
above).

Modes that no longer record anything are named `UNUSED_<n>`, where `<n>` is
the mode's number. The numbers are unchanged, so older logs still decode,
but a diff that sets one of the old names is rejected as an invalid value.
Set `debug_mode = NONE` instead.

| Number | Old name |
|---|---|
| 8 | `RC_SETPOINT` |
| 25-27 | `RANGEFINDER`, `RANGEFINDER_QUALITY`, `LIDAR_TF` |
| 29 | `GOVERNOR` |
| 30 | `SDIO` |
| 33 | `SMARTAUDIO` |
| 34 | `RTH` |
| 36 | `ACRO_TRAINER` |
| 46-47 | `TTA`, `AIRBORNE` |
| 54 | `GPS_RESCUE_THROTTLE_PID` |
| 56-57 | `FEEDFORWARD_LIMIT`, `FEEDFORWARD` |
| 61-62 | `D_LPF`, `VTX_TRAMP` |
| 69-77 | `PITCH_PRECOMP`, `YAW_PRECOMP`, `RESCUE`, `RESCUE_ALTHOLD`, `CROSS_COUPLING`, `ERROR_DECAY`, `HS_OFFSET`, `HS_BLEED`, `GOV_MOTOR` |
| 79-81 | `AUTOHOVER`, `ATTHOLD`, `TVHOLD` |
| 83-84 | `SNAP_RELAX`, `PROP_HANG` |

## Servos

There are 24 servos. Servo N is sent on PWM output N, where the board has
one, and on bus channel N (see `fbus_master_channels` and
`sbus_out_channels` above, and
the [Servos tab](../configurator/tabs/servos.md);
channels past the count are sent at centre). The `servo` command lists the
servos in use and sets one:

```
servo <index> <mid> <min> <max> <rneg> <rpos> <speed> <flags>
```

| Argument | Range | Default | Meaning |
|---|---|---|---|
| `<index>` | 1 to 24 | | The servo |
| `<mid>` | 1000 to 2000 | 1500 | Center, µs |
| `<min>` | -1000 to 0 | -600 | Lower travel limit, µs from center |
| `<max>` | 0 to 1000 | 600 | Upper travel limit, µs from center |
| `<rneg>`, `<rpos>` | 50 to 1000 | 500 | Scale on the negative and positive side, µs |
| `<speed>` | 0 to 60000 | 0 | Speed limit, ms per 60°; 0 is no limit |
| `<flags>` | | 0 | Bit 0 reverse, bit 1 geometry correction |

The signal sent to a servo stays within 880-2159 µs. `servo status`,
`servo override`, `servo trim` and `servo name` use the same index.

`servo name` lists the servos that have a name, `servo name <index> <name>`
sets one and `servo name <index>` clears it. A name is up to 8 printable
characters and may contain spaces:

```
servo name 1 STEER
servo name 2 REAR STR
```

Names are labels only; nothing reads them while driving. `diff` and `dump`
include them.

!!! note "Backups from before MSP API 22.16"
    The `servo` line had a ninth value, a rate per servo, between `<rpos>`
    and `<speed>`. Such a line is still accepted and its rate is ignored.
    `bus_servo_clone_pwm` no longer exists. A `servo`, `servo trim` or
    `mixer rule` line that names servo 9 or above meant a bus channel 8
    lower than it does now, so don't paste those lines back as they are.

## Mixer rules

```
mixer rule <index> <operator> <input> <output> <weight> <offset> <weight-neg> <speed> <curve> <condition> <role>
```

Everything after `<offset>` is optional: `<weight-neg>` defaults to the
symmetric value, and the rest default to off. `<operator>` is `set`, `add`
or `mul`; `mixer rule <index> del` deletes a rule and `mixer rule reset`
goes back to the default rules. `dump` and `diff` print rules in this form,
with names for the input and output. The default rules are the car preset
(see [Car Mixer, Channels and Gyro Direction](../driving/car-mixer.md)):

```
mixer rule 0 set SY S1 1000 0 1000 0 0 0 0
mixer rule 1 set ST M1 1000 0 1000 0 0 0 0
```

Stabilized steering (`SY`) drives servo 1 and stabilized throttle (`ST`)
drives motor 1. The [drive aids](../driving/drive-aids.md) work on the
stabilized throttle, so an ESC rule fed from the RC throttle (`CT` or `RT`)
bypasses them.

`<output>` is a number or a name:

| Output | Name | Meaning |
|---|---|---|
| 0 | `-` | None |
| 1-24 | `S1`-`S24` | Servos 1-24 |
| 25-26 | `R25`, `R26` | Reserved |
| 27-30 | `M1`-`M4` | Motors 1-4 |

`<input>` is a number or a name:

| Input | Name | Meaning |
|---|---|---|
| 0 | | None |
| 1-4 | `SR`, `SP`, `SY`, `ST` | Stabilized roll, pitch, yaw (steering), throttle. Roll and pitch are always zero |
| 5-8 | `CR`, `CP`, `CY`, `CT` | RC command (stick deflection after the deadband) roll, pitch, yaw, throttle |
| 9-12 | `RR`, `RP`, `RY`, `RT` | RC channel roll, pitch, yaw, throttle |
| 13-26 | `AUX1`-`AUX3`, `CH8`-`CH18` | RC channels 5-18 |
| 27-29 | `TR`, `TP`, `TY` | Reserved (thrust vectoring removed), always zero |
| 30-35 | `CH19`-`CH24` | RC channels 19-24 |

RC channels here are in the firmware's own order (roll, pitch, yaw,
throttle, then AUX), after the `map` command has been applied, not the
order on the receiver. With the default car map `RT1234AE`, CH1 is steering
(yaw), CH2 throttle, CH3-CH6 AUX1-AUX4 and CH7/CH8 roll and pitch.

The optional last argument, `<role>`, is a label for the Configurator: `0`
none (the default when omitted), `1` Flap Compensation, `2` Differential
Thrust Yaw. Both named roles are inherited from WingFlight; the mixer never
reads the role. See [Mixer](../configurator/tabs/mixer.md).
