# Tools and Logs

## Tools

![The Tools menu](../assets/images/ethos-suite/tools_menu.png)

### Setup Wizard

The radio's copy of the Configurator's car [Setup Wizard](../getting-started/car-setup.md):
the same steps in the same order, as a guided sequence for a radio screen,
so a car can be set up at the track without a computer.

!!! warning "Wheels off the ground"
    Put the car on a stand with its wheels off the ground. The wizard turns
    the steering itself, and the ESC step runs the motor forward and in
    reverse. While the wizard is open the controller is in its setup state,
    which blocks arming.

The steps are: Sensors and level, Receiver, Servo bus, Model, Servo type,
ESC, ESC telemetry, Center, Steering direction, Binding limits, Gyro
direction, Trim and gain, Mode switches (ARM, GYRO OFF, DRIVE AIDS OFF,
LAUNCH, DRIFT), Driving style and Finish. A checklist button in the header lists them all
and jumps to any one.

What differs from the Configurator:

- **Receiver** only checks steering and throttle and can set the car
  channel order (CH1 steering, CH2 throttle). The receiver's port and
  protocol can only be set in the Configurator, because the radio talks to
  the controller through the receiver.
- Like the Configurator's, it works only on steering servos (the servos the
  mixer drives from the stabilised steering) and names them **STEER**, **F
  STEER** / **R STEER** for a pair, or **STEER n** for more. The Steering
  direction step reverses any servo that moves the wrong way.
- Changes reach the car as you make them and are kept only once saved.
  Leaving the wizard with unsaved changes asks whether to **Save** or
  **Discard** them; closing the tool can't ask, so it leaves them on the
  controller, unsaved.
- Steps whose settings need a restart (servo bus, servo type, ESC protocol,
  ESC telemetry, board orientation) end with **Save and restart**, and the
  wizard reopens on the same step.
- Everything the wizard holds (overrides, the setup state, the motor run)
  lapses by itself if the radio is switched off or the link drops.

### Copy Profiles

Copies one PID or rate profile over another. Pick the profile type, the
source and the destination, then press **Save**. Copying a PID profile also
copies its drive aids settings.

![Copy Profiles page](../assets/images/ethos-suite/copy_profiles.png)

### Select Profile

Switches the controller to another PID or rate profile. The tuning pages,
including ABS and Traction, follow the profile you pick.

![Select Profile page](../assets/images/ethos-suite/profile_select.png)

### Diagnostics

Read-only pages for checking the link and the controller:

![Diagnostics menu](../assets/images/ethos-suite/diagnostics_menu.png)

| Page | What it shows |
| --- | --- |
| Status | Whether the radio side is working: free memory, the RF module, the MSP sensor, the connection to the controller and the API version. |
| ELRS Telemetry | Probes the ExpressLRS module and its link. It needs CRSF telemetry. |
| FC Status | The controller's arming flags, blackbox space, load, and active profiles. |
| Rx Backup | The [backup receiver](../driving/backup-rx-input.md) inputs: each backup's protocol and link, which source is active, and their channels side by side. |
| Info | Version numbers: the suite, Ethos, the firmware, and the MSP protocol. |

![Diagnostics, Status page](../assets/images/ethos-suite/diagnostics_rfstatus.png)

![Diagnostics, Info page](../assets/images/ethos-suite/diagnostics_info.png)

When something doesn't work, **Status** and **Info** are the first places to
look, and worth a screenshot when you ask for help.

## Logs

The radio logs telemetry while you drive: voltage, current, motor RPM, ESC
temperature and throttle. **Logs** works without the car connected.

Logs are kept per model, under the name the controller reports:

![Logs: one folder per model](../assets/images/ethos-suite/logs-folders.png)

Inside, runs are grouped by date, one tile per run:

![A model's logs, grouped by date](../assets/images/ethos-suite/logs-list.png)

Open a run to see it as a graph. The legend on the right shows each value's
minimum and maximum over the run, and its value at the cursor. Drag the
slider to move through the run. **-** and **+** zoom out and in; they are
greyed out when the whole run already fits.

![A log graph, with voltage, current, RPM, ESC temperature and throttle](../assets/images/ethos-suite/logs-graph.png)
