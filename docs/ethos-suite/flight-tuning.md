# Flight Tuning

**Flight Tuning** (the suite still uses this aircraft name for it) holds the
settings you change most at the track: the steering gyro's PIDs, the rates,
how the car feels, and the drive aids. The pages are the same settings as
the Configurator's, so each section below links to the Configurator page
that explains them.

![The Flight Tuning menu](../assets/images/ethos-suite/flight-tuning-menu.png)

The menu has three groups: **Flight Tuning** (PIDs, Rates, Flight Feel),
**Drive Aids** (ABS) and **Advanced** (Filters, PID Controller, PID
Bandwidth, Gain Curves, Rate Response).

!!! note "Only steering is stabilised"
    Gripora only stabilises yaw, which is the steering. The suite's tuning
    pages still show a roll, pitch and yaw column, as they did on WingFlight;
    on a car only the **yaw** values do anything. Roll and pitch are held at
    zero by the firmware, whatever the pages show.

Most of these pages belong to a PID profile, shown in the title as
*PIDs #1* and so on. To work on another profile, switch to it (from the radio,
or with [Select Profile](tools-and-logs.md#select-profile)) and the page
follows.

## PIDs

P, I, D, F and B per axis. On a car, use the yaw row: it is the steering
gyro's gains.

![PIDs page](../assets/images/ethos-suite/pids.png)

See [Profiles](../configurator/tabs/profiles.md).

## Rates

RC Rate, Rate and RC Expo per axis, for the active rate profile (the title
shows its number). The yaw rates set how fast the car is asked to turn at
full steering.

![Rates page](../assets/images/ethos-suite/rates.png)

See [Rates](../configurator/tabs/rates.md).

## Flight Feel

Gain, Decay and Relax per axis, plus the overall Throttle and Speed gains.
The Configurator calls this **Driving Feel**. The Speed gain follows the
car's speed from the motor RPM, or from GPS without an RPM source.

![Flight Feel page](../assets/images/ethos-suite/master_gains.png)

See [Profiles](../configurator/tabs/profiles.md).

## ABS

Under the **Drive Aids** group. The timed ABS for the current PID profile
(the title shows it, such as *ABS #1*): **Mode** (Off, Static or Dynamic),
**Trigger**, **Pulse Rate**, **Release Floor** and **Hold**, the **Full
Brake Rate** and **Full Brake Hold** that Dynamic moves towards, and the
**Rate Added at Lock** and **Hold at Lock** that follow the steering. Each
PID profile has its own settings, so a profile switch changes them with the
tune.

A switch for the **DRIVE AIDS OFF** mode (on the Modes page) turns the drive
aids off from the transmitter. ABS is also off while disarmed and in
failsafe.

The suite has no traction control page yet: set traction control in the
Configurator or with the EdgeTX scripts.

See [Drive Aids](../driving/drive-aids.md) for what each setting does and
which ESC setups ABS can run on.

## Filters

The gyro lowpass filters (type, cutoff, and dynamic range) and the notch
filters. The page scrolls.

![Filters page](../assets/images/ethos-suite/filters.png)

See [Gyro](../configurator/tabs/gyro.md).

## PID Controller

How the PID loop handles error: error decay, error limits and I-term relax.
The page scrolls. It still has a **Cross-Axis Relax** group from WingFlight;
the firmware no longer has that feature, so leave it alone.

![PID Controller page](../assets/images/ethos-suite/pid_controller.png)

See [Profiles](../configurator/tabs/profiles.md).

## PID Bandwidth

Cutoff frequencies for the gyro, D-term and B-term, per axis.

![PID Bandwidth page](../assets/images/ethos-suite/pid_bandwidth.png)

## Gain Curves

A gain curve per axis, for throttle and for speed, and the speed range the
speed curve covers. The firmware takes 10 to 300 km/h here (default 60);
the current suite still offers up to 600 km/h, and the firmware limits a
value above 300 to 300.

![Gain Curves page](../assets/images/ethos-suite/gain_curves.png)

See [Profiles](../configurator/tabs/profiles.md) and
[Curves](../configurator/tabs/curves.md).

## Rate Response

How quickly the car follows the steering: response time, acceleration limit,
setpoint boost and its cutoff per axis, and the dynamic ceiling gain.

![Rate Response page](../assets/images/ethos-suite/rates_advanced.png)

See [Rates](../configurator/tabs/rates.md).
