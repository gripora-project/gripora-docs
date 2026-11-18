# Maintenance

The Maintenance tab, in the Diagnostics group, holds the one-off actions on
the controller: calibration, saving, resetting and rebooting.

| Button | What it does |
|---|---|
| **Calibrate Accelerometer** | Put the car (or the board) on a level surface and keep it still while it calibrates. |
| **Calibrate Magnetometer** | Only with a magnetometer: turn the car at least 360° about every axis within 30 seconds. |
| **Save Settings** | Writes the current settings to the controller's memory. |
| **Reset Settings** | Restores every setting to its default, after a confirmation. Take a [backup](../../getting-started/backup-and-restore.md) first. |
| **Boot Loader / DFU** | Reboots into the boot loader, for flashing firmware. |
| **Mass Storage Mode** | Reboots so the controller shows up as a USB drive, for copying Blackbox logs. |
| **System Reboot** | Reboots and reinitialises the controller. |

The accelerometer is used for the board orientation and for which way is
level. The Setup Wizard's first step calibrates it too, see [Set Up Your Car](../../getting-started/car-setup.md).

A reset sets the car defaults: steering servo on S1, ESC on M1, CH1
steering and CH2 throttle. See [Car Mixer and Channels](../../driving/car-mixer.md).
