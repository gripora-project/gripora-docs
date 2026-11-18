# First Connection

Once firmware is flashed, connect to the controller from the main
Configurator window (not the Firmware Flasher tab).

## Connecting

1. Plug in the controller via USB (or connect over its configured
   serial link, e.g. Bluetooth).
2. Select the correct port in the port picker at the top of the window.
3. Click **Connect**.

## If the firmware version isn't recognized

If the Configurator can't validate the firmware version or type it just
connected to -- for example, very old/incompatible firmware, or a board
that isn't running Gripora firmware at all -- it shows a warning dialog
and falls back to **CLI mode**, which works independent of the normal MSP
configuration protocol. This lets you back up settings or investigate over
the CLI even when the full Configurator UI can't be used yet.

## What to check first

Once connected, the **[Status](../configurator/tabs/status.md)** and
**[Sensors](../configurator/tabs/sensors.md)** tabs are the first useful
stops -- confirm the gyro/accelerometer are live and the receiver channels move
before moving on to the [Setup Wizard](car-setup.md), which finds the board
orientation and sets up the mixer, servos and ESC.
