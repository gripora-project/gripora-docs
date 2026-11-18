# Setup

**Setup** covers how the car is built and wired: the board and radio, the
sensors, the mixer and servos, and power and the ESC. Each section links to
the Configurator page that explains the settings.

![The Setup menu, top half](../assets/images/ethos-suite/setup-menu.png)

![The Setup menu, scrolled down to Mixer & Servos and Power & Motors](../assets/images/ethos-suite/setup-menu-2.png)

For a first setup from the radio, use the step-by-step
[Setup Wizard](tools-and-logs.md#setup-wizard) under Tools instead. The pages
here are for changing one thing later.

## Board & Radio

### Configuration

The car's name (the page still labels it *Craft name*), PID loop speed, and
the features the board runs: LED strip and the ready-to-arm wiggle. There is
no GPS switch: GPS runs when the GPS source matches a port (see
[GPS](../configurator/tabs/gps.md)).

![Configuration page](../assets/images/ethos-suite/configuration.png)

See [Configuration](../configurator/tabs/configuration.md) and
[Ready-to-Arm Wiggle](../driving/ready-to-arm-wiggle.md).

### Ports

What each UART is used for, and its speed.

![Ports page](../assets/images/ethos-suite/ports.png)

See [Serial Ports](../reference/serial-ports.md).

### Radio Config

Stick deflection and centre, the throttle range, and the deadbands. On a car
the throttle is signed around the centre: forward above it, brake and
reverse below.

![Radio Config page](../assets/images/ethos-suite/radio_config.png)

See [Receiver](../configurator/tabs/receiver.md).

### Telemetry

The telemetry sensors the controller sends, grouped by kind. Open a group to
turn sensors on or off. **Tool** selects the default sensors.

![Telemetry page](../assets/images/ethos-suite/telemetry.png)

See [Receiver](../configurator/tabs/receiver.md) and
[Telemetry](../reference/telemetry.md).

### Controls

A menu of its own:

![Controls menu](../assets/images/ethos-suite/controls_menu.png)

| Page | What it is for | More |
| --- | --- | --- |
| Modes | Switch ranges for each mode, such as ARM, GYRO OFF and DRIVE AIDS OFF. Pick a mode, then add or change its ranges. | [Modes](../configurator/tabs/auxiliary.md) |
| Adjustments | Adjustments from a switch or knob while driving, such as a gain knob. | [Adjustments](../configurator/tabs/adjustments.md) |
| Failsafe | **Ch. Fallback**: what each channel does when the signal is lost. **Stage 2**: the failsafe procedure and its delays. | [Failsafe](../configurator/tabs/failsafe.md) |
| Beepers | Which events sound the beeper, and the ESC beacon. | [Beepers](../configurator/tabs/beepers.md) |
| Blackbox | The logging device, rate and fields, and the log memory's status. **Tool** on *Status* erases it. | [Blackbox](../configurator/tabs/blackbox.md) |
| Stats | Run count, and last and total run time (labelled *Flight Count* and *Flight Time* in the suite). | |

![Modes page](../assets/images/ethos-suite/modes.png)

![Failsafe, Stage 2 page](../assets/images/ethos-suite/failsafe_procedure.png)

## Sensors & GPS

### Accelerometer

Accelerometer trim for roll and pitch. **Tool** calibrates the
accelerometer: keep the car level and still while it does.

![Accelerometer page](../assets/images/ethos-suite/accelerometer.png)

### Alignment

How the board is mounted, with a live 3D view of the car that follows the
board as you move it. **Tool** turns the view so the rear of the car faces
you.

![Alignment page](../assets/images/ethos-suite/alignment.png)

See [Configuration](../configurator/tabs/configuration.md).

## Mixer & Servos

### Mixer

Gain and direction for each stabilised axis. On a car only yaw (the
steering) is used. The car mix itself (steering on S1, the ESC on M1,
optional rear steering and a second ESC) is set by the Setup Wizard's Model
step, or in Mixer Rules.

![Mixer page](../assets/images/ethos-suite/mixer_config.png)

See [Mixer](../configurator/tabs/mixer.md) and
[Car Mixer and Channels](../driving/car-mixer.md).

### Mixer Rules

The mixer's rules, one row each. Select a rule to edit it.

![Mixer Rules page](../assets/images/ethos-suite/mixer_rules.png)

### Curves

Mixer, gain and servo curves.

![Curves menu](../assets/images/ethos-suite/curves.png)

See [Curves](../configurator/tabs/curves.md).

### Servos

Lists the servos in use, whether they are on a PWM output or a bus; select
one to set its centre, limits and direction. **Tool** turns servo override
on, so a servo moves to its centre as you change it. The PWM update rate
isn't set here: use the Configurator's
[Servos](../configurator/tabs/servos.md) tab or the wizard's Servo type
step.

![Servos](../assets/images/ethos-suite/servos.png)

See [Servos](../configurator/tabs/servos.md).

## Power & Motors

### Power

| Page | What it is for |
| --- | --- |
| Battery | Battery profiles: capacity and cell voltages. |
| Alerts | Run time alarm (labelled *Flight Time Alarm*) and the BEC and receiver voltage alerts. |
| Sources | Where voltage and current are measured. |
| SmartFuel | How fuel (charge left) is worked out. |
| Preferences | The ESC current limit. |

![Power, Battery page](../assets/images/ethos-suite/power_battery.png)

See [Power](../configurator/tabs/power.md).

### ESC & Motors

![ESC & Motors menu](../assets/images/ethos-suite/esc_motors_menu.png)

| Page | What it is for | More |
| --- | --- | --- |
| Motor Override | Run the motor from the radio, for testing. Put the car on a stand with the wheels off the ground first. | |
| Throttle | Throttle protocol, update rate and PWM endpoints. | [Motors](../configurator/tabs/motors.md) |
| Telemetry | ESC telemetry protocol and corrections. | [Motors](../configurator/tabs/motors.md) |
| SRXL2 ESC | Shown only while the ESC telemetry protocol is SRXL2. | [Motors](../configurator/tabs/motors.md) |
| RPM | Where RPM comes from, the gear ratios and the motor pole count. | [Motors](../configurator/tabs/motors.md) |
| ESC Prog. | Change your ESC's own settings from the radio, through the controller. Pick the ESC's make. | [ESC Programming](../configurator/tabs/esc-programming.md) |

The RPM page still labels its two ratios *main motor* and *tail motor*. On a
car they are the gear ratios (motor turns per wheel turn) of motor 1 and
motor 2, which the controller uses to work out the car's speed.

The suite has no page for the drive settings (ESC behaviour below neutral,
tyre diameter, the gyro start speeds and the reverse gyro). Set those on the
Configurator's [Motors](../configurator/tabs/motors.md) tab; the Setup
Wizard's ESC step sets the ESC behaviour. See
[Drive State](../driving/drive-state.md).

![Motor Override page](../assets/images/ethos-suite/motor_override.png)

![ESC Prog. menu](../assets/images/ethos-suite/esc_forward_menu.png)

Each ESC page shows the ESC it found, then its settings in groups:

![ESC Prog., AM32](../assets/images/ethos-suite/esc_forward_am32.png)
