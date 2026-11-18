# XACT Servo Programming

The XACT Servo Programming tab programs FrSky XACT servos on an F.Bus line
from the Configurator. XACT servos are configured over the same F.Bus wire
that drives them, so no separate programming connection is needed. The tab
only appears when a serial port is set to **Bus: F.BUS Master** on the
[Configuration](configuration.md) tab.

For wiring and the whole setup of XACT steering servos, see
[FrSky Servos](../../setup/frsky-servos.md).

## Programming a servo

1. **Scan for Servos** finds the XACT servos on the bus and lists them with
   their Physical ID, App ID and channel.
2. Select one to edit it. The settings are:
    - **Physical ID**: the servo's bus address (00-1A). Every servo on the
      bus needs its own.
    - **App ID**: tells servos of the same model apart. Give each servo on
      the bus a different value.
    - **Channel**: the bus channel (CH1-CH24) the servo follows. Bus channel
      N carries servo N, so the steering servo on S1 listens to CH1.
    - **Range** (120°, 90° or 180° of total travel), **Direction**,
      **Pulse Type** (leave it at 1500 µs), **Data Rate** and **Center**.
    - Advanced: **Holding Strength**, **Operation Smoothing** and
      **Deadband**. Series 65 servos (firmware 40 and later) add **Working
      Mode** and **Max Angle**.
3. **Save** writes the values to the servo.

!!! warning "One servo at a time"
    Servos ship with the same IDs. Two servos sharing a Physical ID collide
    on the bus and show as one, and a save addressed by App ID can reach
    both. The tab warns about duplicates and blocks a save that would keep
    a duplicate App ID. Connect each new servo on its own, give it its own
    IDs, then join it to the bus.

The servo's settings are stored in the servo itself, not in Gripora, so
they are not part of a Gripora backup or `diff`.

## Boards with an FBUS mux

On boards that switch their outputs between PWM and F.Bus with an FPGA, an
**FBUS mux outputs** section lets you put F.Bus on one output alone, so a
scan or write only reaches the servo plugged in there. Arming is blocked
while one output is selected. **Auto program** gives the servo on each
F.Bus output its own Physical ID, App ID and channel and sets its range to
180°.
