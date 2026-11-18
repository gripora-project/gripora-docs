# FrSky Servos

FrSky XACT servos can run on a single **F.Bus** wire. That one wire carries
the steering command from the controller to the servos and their telemetry
back. Power comes from the servo ports as usual.

This guide covers the bus wiring, programming each servo, forwarding its
telemetry to the radio, and the checks before the first run, for a car with
an XACT steering servo (and optionally an XACT rear steering servo). The
settings themselves are explained on their own pages, linked below.

!!! danger "Wheels off the ground"
    Put the car on a stand with the wheels off the ground, or keep the motor
    disconnected, until every servo has passed the checks in
    [section 6](#6-checks-before-the-first-run).

!!! warning "Program one servo at a time"
    FrSky servos ship with the same Physical ID (`0A`, sometimes `0C`) and
    channel `CH1`. Two servos with the same ID on the bus answer each
    other's telemetry requests, and a save can overwrite all of them at
    once. Connect each new servo alone, assign its ID and channel, save it,
    and only then join it to the bus.

## What you need

- A controller with a free UART that can be set to **Bus: F.BUS Master**
  (function `FBUS_OUT`, see [Configuration](../configurator/tabs/configuration.md)).
- One XACT servo for the steering, and a second for rear steering if the car
  has it.
- Either the Configurator's [XACT Servo Programming](../configurator/tabs/xact-servo.md)
  tab, or a transmitter with the FrSky Ethos XAct menu, to program each
  servo.

## 1. Wiring

Each servo uses two connections:

- **Power** comes from a servo port on the controller, through the usual
  red (+) and black (−) lead: the steering servo on **S1**, a rear steering
  servo on **S2**.
- **Signal** is the F.Bus wire, from the TX pin of the UART set to F.Bus
  master. With more than one servo, the yellow signal wires join at one
  point fed from that pin.

![FrSky servo wiring: power from the servo ports, one F.Bus signal wire from the bus port to every servo](../assets/images/frsky-servos/frsky-servo-wiring.png)

The wiring picture comes from a four-servo aircraft setup on a VANTAC RF007
TWIN; a car uses one or two of the servos. On that board only pin 1 (TX4)
of Port A carries F.Bus, and its other Port A pins must not power a servo.
Check your own board's pinout before wiring.

Use the same plugs and servo leads you would for PWM. The signal pin of the
servo ports is not used, because the servos take their command from the
F.Bus wire.

## 2. Programming the servos

Each servo needs two addresses, set one servo at a time:

| Servo | Use | Channel | Physical ID (hex) | Physical ID (CLI, decimal) |
| :-- | :-- | :-- | :-- | :-- |
| 1 | Steering (front) | `CH1` | `0A` | 10 |
| 2 | Rear steering | `CH2` | `0B` | 11 |

- **Channel** says which bus channel the servo answers to. Bus channel N
  carries servo N (see [Bus Servos](../configurator/tabs/servos.md#bus-servos)),
  so channel `CHn` drives the servo on output `Sn`.
- **Physical ID** is the unique bus address used for telemetry. The valid
  range is `00` to `1A`.

!!! warning "Check the total travel and the horn angle before installing"
    The firmware accepts 120°, 90° and 180° of total travel for XACT
    servos. A Rotorflight test on a helicopter found that with 120° the
    angles did not match the servo positions, and that 180° matched; this
    hasn't been checked on a car. Set the total travel to what your steering
    linkage needs, and check the horn angle at center and at both ends
    **before** the servo is installed. End stops are described in
    [Servos](../configurator/tabs/servos.md#endpoints-and-scale).

### 2.1 Configurator

With a port set to F.Bus master, the
[XACT Servo Programming](../configurator/tabs/xact-servo.md) tab scans the
bus, lists the servos and edits their Physical ID, App ID, channel, range,
direction and the other servo settings. Program each new servo with only
that servo connected.

### 2.2 Transmitter (Ethos)

FrSky's XAct Lua script adds a servo menu to Ethos radios under **System →
Device config → Servos → XAct**. Install it with the FrSky Suite (2.0 or
later), under **Lua library**. It programs a servo connected directly to
the transmitter or receiver. Program each servo this way, one at a time,
and only then join it to the bus.

![Ethos Device config: the Servos category with the XAct entry](../assets/images/frsky-servos/frsky-servos-ethos-device-config.png)
*Ethos Device config → Servos → XAct (German UI).*

The values used on the test servos, as a starting point:

| Setting | Value |
| :-- | :-- |
| Physical ID | `0A`, `0B`, one per servo |
| Application ID | `6800` (leave unchanged) |
| Data rate | `1000 ms` |
| Max. travel, total | see the warning in section 2 |
| Direction | `Clockwise` |
| Working range | `Angle mode` (not continuous rotation) |
| Channel | `CH1`, `CH2` |
| Channel center | `0` |
| Holding force | `0.4` |
| Smoothing | `1` |
| Dead zone | `3` |

![Ethos XAct menu: connection type S.Port, physical ID 0A, application ID 6800, servo firmware 4.1.0, data rate 1000 ms](../assets/images/frsky-servos/frsky-servos-ethos-id-setup.png)
*Ethos XAct menu, read from the servo (German UI).*

![Ethos XAct parameters: direction, channel, centre and holding force](../assets/images/frsky-servos/frsky-servos-ethos-parameters.png)
*Ethos XAct parameters, read from the servo (German UI). The photo shows a
120° setting; use the value from the warning in section 2.*

With the servo on the controller's F.Bus master line and forwarded (section
4), Ethos's Device config pages can also reach it through an F.Port2 or
FBUS receiver; see [FBUS/S.Port Sensors](../configurator/tabs/fbus-sensors.md#configuring-forwarded-devices-from-the-radio).

## 3. Serial port in the Configurator

1. Open the [Configuration](../configurator/tabs/configuration.md) tab.
2. Set the UART that carries the servos to **Bus: F.BUS Master**.
3. Save and reboot.

The Setup Wizard's **Servo bus** step does the same, and finds the right pin
and polarity for you. The servos are read by the master and listed on the
[FBUS/S.Port Sensors](../configurator/tabs/fbus-sensors.md) tab. Each servo
should appear there with its Physical ID once it answers on the bus.

## 4. Forwarding servo telemetry to the radio

The controller forwards a sensor's telemetry to the radio only when it is
set to **Forwarded**. Up to **8 sensors** can be forwarded at once. Out of
the box, none are.

1. Set the **receiver** protocol to F.Port, F.Port2 or FBUS on the
   [Receiver](../configurator/tabs/receiver.md) tab. The Forwarded toggle is
   only enabled for these protocols, because forwarding uses the receiver's
   telemetry return channel.
2. On the [FBUS/S.Port Sensors](../configurator/tabs/fbus-sensors.md#forwarding)
   tab, turn **Forwarded** on for the servos (`0A`, `0B`). The change takes
   effect at once, without a reboot.

The same list can be set in the CLI with `fbus_master_forwarded_sensors`.
Decimal IDs `10, 11` are the servos `0A` and `0B`; `255` is an unused slot:

```text
set fbus_master_forwarded_sensors = 10, 11, 255, 255, 255, 255, 255, 255
save
```

Check the result with the `fbus_sensors` command and its **Forwarded**
column.

On the transmitter, run **Model → Telemetry → Discover sensors**. The radio
then lists the servo's sensors:

![Ethos telemetry page with the servo sensors SRV1: current, voltage, position, velocity and temperature](../assets/images/frsky-servos/frsky-servos-ethos-telemetry-sensors.png)

*Ethos telemetry page after discovery, with live values from the servo
(German UI).*

The controller itself decodes three values from the servo frame: current,
voltage and temperature.

## 5. Bus servos and the mixer

The default mixer is the car setup: the steering on **S1** (Stabilized Yaw)
and the ESC on **M1** (Stabilized Throttle). With rear steering on, the rear
servo is on **S2**. See [Mixer](../configurator/tabs/mixer.md) and
[Car Mixer and Channels](../driving/car-mixer.md).

Bus channel N always carries servo N, so bus channel 1 carries the steering
and bus channel 2 the rear steering, without any extra rule. The
[Servos](../configurator/tabs/servos.md#bus-servos) tab lists them in the
one servo table.

## 6. Checks before the first run

Check the horn angles as the warning in
[section 2](#2-programming-the-servos) describes. Then, with the wheels off
the ground:

1. **Steering direction.** Turn the steering wheel on the radio: the wheels
   must steer the same way. Correct a reversed servo with **Reverse** on the
   Servos tab, not by changing the rule.
2. **Gyro direction.** Turn the car by hand about its vertical axis: the
   steering must turn against the movement. If it turns with it, switch on
   **Reverse gyro (steering)** on the Mixer tab.
3. **Full lock.** Turn to full lock each way and watch the servo current in
   telemetry. A large rise at one point means the steering binds. Reduce the
   travel with Min/Max on the Servos tab.
4. **Power-on.** Power on with all servos connected. They must move to
   center smoothly and must not jerk.

## Troubleshooting

| Problem | Try |
| :-- | :-- |
| Servos do not respond | Check that the UART is set to F.Bus master, the single signal wire, and that each servo is powered from its servo port. |
| Two servos appear as one | Two servos share an ID. Re-program them one at a time. |
| Servo is listed on FBUS/S.Port Sensors, but **Forwarded** is off | Turn **Forwarded** on. The receiver must be F.Port, F.Port2 or FBUS. |
| Telemetry shows nothing on the radio | Run **Discover sensors** on the transmitter again. |

## Not verified for Gripora

- The Port A pin mapping and the warning about its other pins. They come from
  the RF007 TWIN drawing, not from a Gripora board configuration.
- The cause of the 120° versus 180° finding. It comes from a helicopter test
  with Rotorflight and has not been checked against Gripora's servo angle
  mapping.
- This guide has not been tested on a car.
