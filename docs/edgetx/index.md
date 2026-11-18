# EdgeTX Lua Scripts

The Gripora Lua scripts for EdgeTX and OpenTX put the controller's settings
on your transmitter. At the track you can change settings, hear what an
adjustment did, and see telemetry and run statistics in widgets, without a
computer. For FrSky Ethos radios, use the
[Ethos Lua Suite](../ethos-suite/index.md) instead.

!!! note "Being converted to cars"
    The scripts came from WingFlight's along with the firmware. The tuning
    pages label the steering rows *Steer* or *Steering* and mark roll and
    pitch *(unused)*, but some aircraft pages and settings that Gripora no
    longer uses are still there (see
    [Pages and settings left over from WingFlight](#pages-and-settings-left-over-from-wingflight)).

## What they install

- **The Gripora tool** (`gr.lua` in `/SCRIPTS/TOOLS`), opened from the
  radio's *Tools* menu. Its pages read settings from the controller over
  telemetry, let you change them, and save them back.
- **The GR Tool and GR Stats widgets**, on colour radios. *GR Tool* shows the
  name of the connected car and one telemetry value of your choice, and runs
  the background script for you. *GR Stats* shows run statistics.
- **A background script** (`grbg.lua`), optional: it sets the controller's
  clock from the radio (so Blackbox logs get the right time), turns on the
  Gripora telemetry sensors over CRSF/ELRS custom telemetry, and runs the
  *Adjustment Teller*, which says what adjustment you just made.

## What you need

- EdgeTX 2.5.0 or OpenTX 2.3.12 or later. The *GR Tool* widget needs a colour
  radio running EdgeTX 2.11 or later.
- A receiver that supports remote configuration:
    - a FrSky Smartport or F.Port receiver, or
    - a Crossfire v2.11 or newer receiver, or
    - an ELRS 2.0.1 or newer receiver. With ELRS, set the baud rate to 1.87M
      or higher in the radio's *Hardware* menu.
- Gripora firmware. The scripts check the firmware's MSP API version when
  they connect and refuse one they don't support.

## Installing

- **With the updater:** the
  [Gripora EdgeTX/OpenTX Lua Updater](https://github.com/gripora-project/gripora-lua-edgetx-updater/releases)
  downloads the scripts and copies them to the radio's SD card.
- **By hand:** download the latest release from
  [GitHub](https://github.com/gripora-project/gripora-lua-edgetx/releases)
  and copy the contents of its `SCRIPTS` folder to the `SCRIPTS` folder on
  the radio's SD card, either with the radio connected by USB as a drive or
  with the SD card in a computer.

When it is right, `gr.lua` is in `/SCRIPTS/TOOLS` and *Gripora* is listed in
the radio's *Tools* menu. The first start shows *Compiling...* for a while;
that is normal.

To run the background script, either add the *GR Tool* widget to a screen
(colour radios with EdgeTX 2.11 or later), or set `grbg` as a special or
global function that runs when the model is selected (on EdgeTX, with
*Repeat* on).

## Using the tool

Open *Gripora* from the *Tools* menu, or put the *GR Tool* widget in full
screen mode. Nothing is written to the controller until you choose *Save*:
on colour radios it is the button in the top right corner, on black and
white radios it is in the menu you get by long-pressing the wheel.

## Pages

Pages shown by default:

| Page | What it is for |
| --- | --- |
| Status | The current PID and rate profiles, load, the arming disabled flags and the dataflash free space (with an erase button). |
| Rates | The rates. On a car only the *Steer* row is used. |
| Rate Dynamics | Response time, acceleration limit and setpoint boost, and the *Steering Dynamics* ceiling and deadband. Only the steering values are used. |
| PID Gains | P, I, D, F and B per PID profile. On a car only the *Steer* row is used. |
| PID Controller | I-term relax level, I-term decay limit, error limits and the steering bandwidth and D- and B-term cutoffs. |
| Profile - Various | Throttle and GPS speed gain attenuation, *Driving Feel* (master gain, I-term decay and I-term relax), gain curves and the speed range. It still lists aircraft settings too (see below). |
| Battery, Smart Fuel | Battery profiles and the fuel estimate. |
| Servos | Servo centre, limits and direction. |
| Mixer | Gain and direction of the stabilised axes. |
| Gyro Filters | The gyro filters. |
| Accelerometer Trim | Accelerometer trim. |
| ESC Sensor | ESC telemetry settings. |
| Drive Aids | ABS, traction control and launch control for the current PID profile (see below). |
| Model | The model ID, the statistics the controller keeps (total runs, time and distance) and radio settings. |
| Settings | Which pages to show, and the background script options. |

*Mixer Inputs*, *Mixer Rules*, *Experimental* and the ESC programming pages
(AM32, BLHeli_S, Bluejay, FLYROTOR, Hobbywing Platinum V5, Scorpion
Tribunus, XDFly/OMP/ZTW, YGE) are hidden until you turn them on under
*Settings*.

### Drive Aids

The page has the PID profile switcher of the other profile pages, because
the firmware keeps the drive aids per PID profile.

- **ABS**: *Mode* (Off, Static, Dynamic), *Trigger*, *Rate*, *Floor*,
  *Hold*, *Dyn rate* and *Dyn hold* (Dynamic only), *Corner rate* and
  *Corner hold*.
- **Traction**: *Mode* (Off, Ramp) and *Ramp* time.
- **Launch**: *Mode* (Off, Switch), *Start throttle* and *Ramp time*.

See [Drive Aids](../driving/drive-aids.md) for what each setting does.

### Pages and settings left over from WingFlight

These are still in the scripts but do nothing on Gripora, or are refused by
the firmware. Leave them alone; they are due to be removed:

- **Tune Advisor**: the firmware no longer has the tune advisor. The page
  is shown by default; turn it off under *Settings*.
- On **Profile - Various**: *Trainer*, *Angle Mode*, *Cross-Axis Relax*,
  and *Snap Relax* and *Prop Hang* where shown. The firmware keeps these
  fields only as placeholders.
- On **PID Controller**: *Attitude Hold*.
- The **roll** and **pitch** rows and columns, marked *(unused)* where there
  is room: Gripora stabilises only the steering.

## Adjustment Teller

Turn it on under *Settings → Grbg Options → Adjustment Teller*. It reads the
adjustment function and value from telemetry:

- S.Port/F.Port: the sensors 5110 and 5111 must be discovered.
- CRSF/ELRS custom telemetry: include the *Adjustment Function* sensor.

The scripts' README also describes a CRSF setup through the *FM* sensor and
a `crsf_flight_mode_reuse` CLI setting. Gripora firmware has no such
setting, so use CRSF custom telemetry instead.

Some black and white radios don't have enough memory for both the
Adjustment Teller and CRSF/ELRS custom telemetry; the screen then freezes at
random.

More detail, including building the scripts from source, is in the
[scripts' README](https://github.com/gripora-project/gripora-lua-edgetx).
