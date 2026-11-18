# Tuning

**Tuning** holds the settings you change most at the track: the steering
gyro's PIDs, the rates, how the car feels, and the drive aids. The pages are
the same settings as the Configurator's, so each section below links to the
Configurator page that explains them.

![The Tuning menu](../assets/images/ethos-suite/flight-tuning-menu.png)

The menu has three groups: **Steering** (PIDs, Rates, Driving Feel),
**Drive Aids** (ABS, Traction, Launch) and **Advanced** (Filters, PID Controller,
PID Bandwidth, Gain Curves, Rate Response).

Gripora stabilises only the steering, so the tuning pages show only the
steering values. The roll and pitch values the controller still stores are
written back unchanged.

Most of these pages belong to a PID profile, shown in the title as
*PIDs #1* and so on. To work on another profile, switch to it (from the radio,
or with [Select Profile](tools-and-logs.md#select-profile)) and the page
follows.

## PIDs

P, I, D, F and B for the steering gyro.

![PIDs page](../assets/images/ethos-suite/pids.png)

See [Profiles](../configurator/tabs/profiles.md).

## Rates

RC Rate, Rate and RC Expo for the steering, for the active rate profile (the
title shows its number). They set how fast the car is asked to turn at full
steering.

![Rates page](../assets/images/ethos-suite/rates.png)

See [Rates](../configurator/tabs/rates.md).

## Driving Feel

**Steering Gain**, **Decay** and **Relax**, plus the overall **Throttle
Gain** and **Speed Gain**, one per line. The Speed gain follows the car's
speed from the motor RPM, or from GPS without an RPM source.

![Driving Feel page](../assets/images/ethos-suite/master_gains.png)

See [Profiles](../configurator/tabs/profiles.md#driving-feel).

## ABS

Under the **Drive Aids** group. The timed ABS for the current PID profile
(the title shows it, such as *ABS #1*): **Mode** (Off, Static or Dynamic),
**Trigger** and **Release Floor**, then a **Rate** and **Hold** for each of
**Pulse**, **Full Brake** (the values Dynamic moves towards; greyed out
unless Mode is Dynamic) and **At Full Lock** (how the pulses change with
steering lock). Each PID profile has its own settings, so a profile switch
changes them with the tune.

A switch for the **DRIVE AIDS OFF** mode (on the Modes page) turns the drive
aids off from the transmitter. ABS is also off while disarmed and in
failsafe.

See [Drive Aids](../driving/drive-aids.md) for what each setting does and
which ESC setups ABS can run on.

## Traction

Traction control for the current PID profile (the title reads *Traction
Control*): **Mode** (Off or Ramp) and **Ramp Time** (50-2000 ms, default
500 ms), which is greyed out unless Mode is Ramp.

See [Drive Aids](../driving/drive-aids.md).

## Launch

Launch control for the current PID profile (the title reads *Launch
Control*): **Mode** (Off or Switch), **Start Throttle** (10-100 %, default
50) and **Ramp Time** (100-5000 ms, default 1000), greyed out unless Mode is
Switch. Put the **LAUNCH** mode on a switch on the Modes page to stage a
launch.

See [Launch control](../driving/drive-aids.md#launch-control).

## Filters

The gyro lowpass filters (type, cutoff, and dynamic range) and the notch
filters. The page scrolls.

![Filters page](../assets/images/ethos-suite/filters.png)

See [Gyro](../configurator/tabs/gyro.md).

## PID Controller

**I-Term Decay Max Rate**, **I-Term Relax Level** and the steering **Error
Limit**. The I-term decay time itself is on Driving Feel.

![PID Controller page](../assets/images/ethos-suite/pid_controller.png)

See [Profiles](../configurator/tabs/profiles.md).

## PID Bandwidth

The steering's gyro, D-term and B-term cutoff frequencies.

![PID Bandwidth page](../assets/images/ethos-suite/pid_bandwidth.png)

## Gain Curves

A gain curve for Steering, Throttle and Speed, and the **Speed Range** the
speed curve covers (10-300 km/h, default 60).

![Gain Curves page](../assets/images/ethos-suite/gain_curves.png)

See [Profiles](../configurator/tabs/profiles.md) and
[Curves](../configurator/tabs/curves.md).

## Rate Response

How quickly the car follows the steering: **Response Time**, **Accel
Limit**, **Setpoint Boost Gain** and **Cutoff**, and the **Dynamic Ceiling
Gain**, **Dynamic Deadband Gain** and **Dynamic Deadband Filter**.

![Rate Response page](../assets/images/ethos-suite/rates_advanced.png)

See [Rates](../configurator/tabs/rates.md).
