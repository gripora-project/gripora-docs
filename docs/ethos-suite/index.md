# Ethos Lua Suite

The Gripora Lua suite puts the controller's settings on your FrSky Ethos
radio. At the track you can retune the steering gyro, the rates and the
drive aids, change setup, read logs and check the car's status without a
computer.

![The Gripora app's main menu on an X20](../assets/images/ethos-suite/root-menu.png)

!!! note "Screenshots predate the car names"
    The screenshots in this section were taken before the suite was renamed
    for cars, so they still show WingFlight, *Flight Tuning*, *Flight Feel*
    and roll, pitch and yaw rows. The text describes the suite as it is now.

The suite installs three things on the radio:

- **The Gripora app**, opened from the radio's System menu. Its pages
  read settings from the controller over telemetry, let you change them, and
  save them back. See [Using the app](using-the-app.md).
- **A dashboard widget** for the model's home screen, showing the arming
  state, battery, fuel, RPM and profile at a glance. See
  [Dashboard](dashboard.md).
- **A background task** that keeps the link to the controller and plays the
  [callouts and alerts](../reference/radio-alerts.md).

## What you need

- A FrSky radio running **Ethos 1.6.2 or later**: X10, X12, X14, X18, X20 or
  Twin X Lite.
- A receiver that carries telemetry to the radio:
    - FrSky, over S.Port or F.Port (ACCESS, ACCST, TD or TW receivers), or
    - ExpressLRS, on a module Ethos supports.
- Gripora firmware with **MSP API 33.0** or later. The suite refuses older
  firmware (including WingFlight firmware) when it connects.
- A suite version that matches your firmware. Each release says which
  firmware it is for.

On an EdgeTX or OpenTX radio, use the
[EdgeTX Lua scripts](../edgetx/index.md) instead.

## Where to go next

1. [Installing](installing.md): put the suite on the radio and turn it on
   for your model.
2. [Using the app](using-the-app.md): getting around, and how saving works.
3. A tour of each part of the app:
    - [Tuning](flight-tuning.md): PIDs, rates, the drive aids
    - [Setup](setup.md): board, radio, mixer, servos, power and ESC
    - [Tools and Logs](tools-and-logs.md): the setup wizard, profiles,
      diagnostics and logs
    - [Settings](settings.md): the suite's own options
4. [Dashboard](dashboard.md): the home-screen widget, and its
   [themes](dashboard-themes.md).

The settings themselves are the same ones the Configurator shows, so each
page in the tour links to the matching Configurator page for what they do.
