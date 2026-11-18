# Status

The Status tab is the first view after connecting. It shows a live summary
of the controller: whether it can arm and, if not, why; the battery; a 3D
car that follows the controller's attitude; and the receiver channels.

Use it as the first check after connecting: a sensor that isn't detected or
a flag that blocks arming shows up here before you go any further.

The **Info** box shows the craft name, firmware, board and whether the
configuration has been changed from the defaults, with the run count and
run time. The **Instruments** box shows heading, pitch and roll; **Reset Z
axis** zeroes the heading shown on the 3D car.

## Enable Arming

**Enable Arming** lets the car arm while the Configurator is connected,
which it normally can't (the **MSP** flag below). The Configurator asks you
to confirm. Never drive the car with the Configurator connected, and put the
car on a stand with the wheels off the ground before you enable it.

## Arming disable flags

When the car won't arm, the Status tab lists each active arming-disable
flag; hover one for a plain-language explanation. The ones you'll meet most
often on a car:

- **THROTTLE**: the throttle isn't at neutral. A car only arms with the
  trigger at neutral (the channel center, with a small deadband).
- **MSP**: the Configurator is connected. Use **Enable Arming** above for a
  bench test.
- **RX_FAILSAFE**: no valid receiver signal.
- **CALIBRATING**: the gyro is still calibrating; keep the car still after
  power-up.
- **ANGLE**: the attitude estimate isn't ready yet.
- **SETUP**: a setup tool, such as the Setup Wizard or a servo, mixer or
  motor override, is holding the outputs. Arming works again once it lets
  go.
- **BACKUP_RX**: a [backup receiver](../../driving/backup-rx-input.md) is
  configured but hasn't linked yet, so the first arm of the session is
  blocked until every configured backup does. Check the backup receiver's
  power, binding and wiring, or power-cycle it, then confirm its **Link**
  badge on the [Receiver](receiver.md) tab comes up.

The full list of flags, and what clears each one, is in
[Arming and Stick Commands](../../reference/stick-commands.md).
