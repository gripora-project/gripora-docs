# Receiver

The Receiver tab sets how the controller talks to your receiver: the
protocol and its signal options, the channel order, the channel range and
deadband, telemetry, and any backup receivers. Live bars show every channel.

## Protocol and signal options

**Receiver Protocol** in the **Primary Receiver** box picks the protocol
(serial protocols such as CRSF, S.BUS, FBUS, SRXL2 and others, or PWM/PPM
and SPI receivers where the board has them). With a serial protocol, three
signal switches appear. Get the wrong one and you see no signal at all,
not a subtle problem:

- **Inverted** flips the signal polarity. Turn it on for F.Port and F.Bus.
  Leave it **off** for Futaba S.BUS and S.BUS2: the controller already
  allows for S.BUS being inverted; turn it on only for a receiver that
  outputs non-inverted S.BUS. Not supported on F4-based boards.
- **Half-Duplex** uses the Tx pin for both sending and receiving. Required
  for SBUS2, F.Port, SRXL and SRXL2. Plain S.BUS leaves it off.
- **Pin Swap** swaps the Rx and Tx pins, for wiring that comes in reversed.
  Not supported on F4-based boards.

**Detect Wiring** tries every combination of the three switches for the
saved protocol and stops at the first one that gives a signal. The
transmitter must be on and bound, the car disarmed, and the tab saved first.
Nothing is kept until you save.

For Futaba receivers, use **Futaba S.BUS2** if the receiver has an S.BUS2
port (control and telemetry on one wire), or **Futaba S.BUS** if it doesn't
(control only). The receiver must be set to output the same one.

For FrSky receivers set to F.Bus or F.Port2, pick **FrSky FBUS**. They are
the same protocol at two speeds, and the controller finds the speed the
receiver uses and keeps it once good frames arrive; telemetry starts after
that. Changing the receiver between F.Bus and F.Port2 needs no change here.

## ExpressLRS switch mode

ExpressLRS receivers use the **TBS CRSF** protocol.

!!! warning
    Gripora only supports the ExpressLRS **Full Res 16ch Rate/2** switch
    mode, at the **100Hz Full** or **333Hz Full** packet rate. It sends all
    16 channels at full (10-bit) resolution, including channel 5, at half
    the packet refresh rate.

    The other switch modes send channel 5 (ExpressLRS AUX1) only as a
    two-position arm switch. Gripora uses channel 5 as a normal
    full-resolution channel (AUX 3 in the car channel order), so in those
    modes it can only be fully one way or the other.

When the protocol is TBS CRSF, the Receiver tab and the Setup Wizard's
receiver step show this warning.

## SRXL2 full-size receivers

Some Spektrum full-size receivers (e.g. the AR6610T) need to see a valid
SRXL2 handshake within about 200 ms of power-on, or they switch their
receiver port to plain PWM instead of SRXL2, which can be too fast for the
normal handshake to win during boot. When the controller's own SRXL2 unit
ID is set to `0` (bus master), it sends the first handshake packets very
early during boot, then carries on with the normal handshake.

This is CLI-only: `set srxl2_unit_id = 0`. The default is `1`, so set it
explicitly for a receiver that needs it.

## Channel Range

- **Channel Center** is the channel value with the steering wheel and the
  throttle trigger at neutral, usually 1500 µs (1520 µs on some Futaba
  radios).
- **Channel Travel** is how far the channel moves from center at full
  steering lock or full throttle.
- **Steering Deadband** is how far the steering channel must move from
  center, in µs, before the car steers (default 5 µs, range 0-100). It keeps
  a wheel that rests slightly off center from steering on its own. The CLI
  setting is `yaw_deadband`. `roll_deadband` and `pitch_deadband` still
  exist in the CLI but have no use on a car.

### Throttle neutral

The throttle is centered like the steering: above neutral drives forward,
below it brakes and reverses. **Throttle Neutral** shows the neutral the
controller uses: the Channel Center, with a small fixed deadband around it.
The car only arms with the throttle at neutral, and a receiver failsafe
returns the throttle to neutral.

## Channel Assignment

**Channel Order** picks which receiver channel drives the steering, the
throttle and AUX 1-4. The default is **Car (CH1 steering, CH2 throttle)**,
as car and pistol-grip radios (Futaba, Sanwa, Spektrum, FrSky, Radiomaster
and most others) send them: CH1 steering, CH2 throttle, CH3-CH6 AUX 1-4.
Pick **Custom order**, or change a channel in the list, if your radio
differs. A custom order is one letter per channel from CH1: **S**
steering, **T** throttle, **1**-**4** AUX 1-4, **-** not used, for example
`TS1234` for throttle on CH1 and steering on CH2. See
[Car Mixer and Channels](../../driving/car-mixer.md).

The live bars next to each channel let you check that the order matches
your radio before you set up the [Mixer](mixer.md): turn the wheel and the
Steering bar moves, pull the trigger and the Throttle bar moves.

Up to 24 channels are shown. Most receiver protocols carry 16 or 18; FrSky
F.Bus in its 24-channel mode carries all 24. Channels beyond the four AUX
channels can be used for modes, adjustments, failsafe and as
[Mixer](mixer.md) inputs. Every channel picker in the Configurator names a
channel **CH #N**, with its role where it has one, for example
`CH #3 (AUX 1)`.

**RSSI** can be read from **AUTO** (reported by the receiver protocol, if
the link supports it), the board's analog RSSI input (**ADC**), or a spare
channel (CH #5 onward) that your receiver puts an RSSI value on.

**Virtual radio** (desktop app only) simulates channel input for bench
testing without a transmitter. **Bind Receiver** sends a bind request to
receivers that support it.

## Telemetry

Telemetry settings for the receiver link: the telemetry signal options and,
for CRSF/ELRS, **Custom Telemetry** (Gripora's own sensor set, decoded by a
Lua script on the radio) with its **Telemetry Packet Rate** and **Telemetry
Packet Ratio**, which must match the ELRS settings on the transmitter.

## Telemetry Sensors

Selects which telemetry values are sent back to your transmitter, and for
protocols with a fixed transmission sequence (e.g. Smart Port), in what
order: values earlier in the list update more often. The sensors on offer
depend on what the protocol and the firmware support (e.g. RPM only appears
with a configured RPM source).

New configurations start with the sensors the Gripora Lua suites use,
including **System Status** and **System Config** in the Status group. These
two carry the controller's status (arm state, receiver links, failsafe,
GPS, profiles and more) for the radio's
[callouts and dashboard alerts](../../reference/radio-alerts.md). See
[Telemetry](../../reference/telemetry.md) for the default list and what each
status bit means.

## Primary Receiver / Backup Receiver 1 and 2

The main receiver's protocol box is labelled **Primary Receiver**. Its title
bar always shows a **Link** badge for the main receiver's own signal,
whether or not a backup port is set up.

If a serial port is set to **Receiver: Backup 1** on the
[Configuration](configuration.md) tab, a second box, **Backup Receiver 1**,
appears below it; a port set to **Receiver: Backup 2** adds a **Backup
Receiver 2** box. Each has:

- A **Protocol** list (None, Futaba S.BUS, FrSky FBUS, FrSky F.PORT, Jeti
  EXBUS or TBS CRSF). `None`, the default, means the port is assigned but
  not decoding anything yet. FrSky FBUS also covers F.Port2 receivers, as on
  the main receiver.
- **Inverted**, **Half-Duplex** and **Pin Swap**, with the same meaning as
  on the main receiver but set separately, since this is a different UART.
  `OFF` always means "this protocol's normal wiring": S.BUS is naturally
  inverted, the others aren't.
- **Link** and **Active Source** badges in its title bar. **Active Source**
  names the receiver in control: Main RX, Backup 1 or Backup 2.

Changes to these fields need **Save and Reboot**.

Once a backup protocol is set and receiving, every channel in
[Channel Assignment](#channel-assignment) gets a smaller bar per backup
under the main one, showing that backup's value for the channel, so you can
check a bound backup gives sane values before relying on it. See
[Backup RX Input](../../driving/backup-rx-input.md) for what the feature
does, including the failover order.
