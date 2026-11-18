# Servos

The Servos tab configures each servo: center point, travel
limits, direction (reverse), speed and trim, on top of whatever mixer
rules send input to it.

There are 24 servos. Servo N is sent on PWM output N, where the board has
one, and on channel N of a bus output (SBUS, F.Bus) when one is set up, so
a servo is set up the same way however it is wired. The table lists the
servos in use; see [Bus Servos](#bus-servos).

Set the limits conservatively at first. It's easier to increase travel once
you've checked the steering doesn't bind at full lock than to find a
stalled, straining servo afterwards. The Setup Wizard's **Binding limits**
step helps with this.

## Name

Each servo can have a short **Name**, such as `STEER`, so you can tell the
servos apart without counting outputs. A name is up to 8 characters
(letters, digits, spaces and punctuation). It is saved on the controller
and kept in a CLI backup, but nothing while driving uses it. The column
only shows with firmware that supports names.

The Mixer tab, the Setup Wizard, Servo Override and Balance show the name
with the servo number, for example *Servo #1 (STEER)*. When a mixer setup
(the Mixer tab's car setup or the Setup Wizard's model step) assigns the
servos, it names them for you: `STEER` for a single steering servo, `F
STEER` and `R STEER` with rear steering. A name it gave earlier to a servo
the new mix no longer uses is cleared; a name you typed is kept.

The Ethos Lua suite shows and edits the names too. The EdgeTX scripts show
them but can't edit them.

## Endpoints and scale

**Min**/**Max** cap the servo's pulse-width travel, to keep the steering off
its mechanical stops at full lock. **Scale Neg**/**Scale Pos** are a
separate pair of per-side scaling factors (500μs by default) -- adjust
these instead of Min/Max specifically to correct a servo that throws
noticeably further on one side than the other.

The limits are the same for every servo, on a PWM output and on a bus:

| Field | Range | Default |
|---|---|---|
| Center | 1000-2000 µs | 1500 µs |
| Min / Max | down to 880 µs / up to 2159 µs | 600 µs either side of Center |
| Scale Neg / Scale Pos | 50-1000 µs | 500 µs |

880-2159 µs is everything an SBUS or F.Bus channel can carry. Narrow-band
servos (760 µs center) are not supported.

## PWM update rate

**PWM update rate**, above the table, is how often every PWM output is
updated. It is one setting for all of them:

- **Standard (50 Hz)** -- the default, and safe for any servo.
- **High speed (333 Hz)** -- responds faster, but is for digital servos
  only. It can damage an analog servo, so check the data sheet first.

It only takes effect after **Save and reboot**, since the rate is set up at
boot time, not applied live like most other fields here. It doesn't affect
bus outputs, which run at their own frame rate. The CLI setting is
`servo_pwm_rate`, 25-333 Hz; a value other than 50 or 333 is shown here as
it is.

## Speed and Reverse

**Speed** limits how fast the servo is allowed to move, expressed as
milliseconds needed for a 60° rotation; 0 means unlimited. It slows one
output, separately from the [Mixer](mixer.md) rule's own Speed limit on the
same output.

**Reverse** flips the direction if the wheels steer the wrong way for the
stick. It reverses the stick and the gyro correction together. If only the
gyro corrects the wrong way, use **Gyro Direction** on the
[Mixer](mixer.md) tab instead, not Reverse here.

## Balance Curve

When two servos drive the same steering (dual steering servos) and don't
quite track each other, a per-servo **balance curve** trims one
servo's travel to match its partner. Balance curves are edited on the
[Curves](curves.md#servo-balance-curves) tab, not here. Any servo with a
non-flat curve shows a curve icon in its row; click it to jump straight to
that servo's curve.

!!! note
    The **Geometry Correction** switch that older Configurator versions
    showed on this tab has been removed from the Configurator, so the
    correction can no longer be turned on or off from here.

## Trim

Trim never changes a servo's **Mid**. Each servo has a separate **Trim**
value in µs, added to its output on top of Mid. Mid and the Min/Max end
stops you set on the bench stay where you put them, however much you trim.

The **Trim** column holds two kinds of trim:

- **Saved trim**: set by a **Stepped** Servo Trim adjustment, or by typing
  it in here. It is saved like any other setting (on disarm, or with
  **Save** on this tab).
- **Live trim**: set by a **Mapped** Servo Trim adjustment (a knob or a
  trim channel from the radio). It follows the channel, is never saved, and
  starts from zero every time the controller boots.

Both together are limited to 20% of the servo's Scale (the larger of Scale
Neg and Scale Pos, so ±100 µs at the default 500 µs). The output also stays
inside the servo's Min/Max, so trim can never drive the steering past the
end stops you set.

How the column shows it:

- **No Servo Trim adjustment for this servo:** the saved trim, editable.
- **A Servo Trim adjustment covers this servo:** read-only, showing the trim
  in use (saved plus live), so it moves as you press the trim. Hover over it
  to see the two parts. A badge beside it shows the adjustment, highlighted
  while active with its channel.

The **Signal** column's full-stick estimate allows for the saved trim: trim
uses up travel on one side and gives it back on the other.

### Clear trims and Move trims into Center

Above the table:

- **Clear trims** sets every saved trim to 0.
- **Move trims into Center** adds each servo's saved trim to its Mid and
  sets the trim to 0. The servo doesn't move, but Min/Max are measured from
  Mid, so the end stops move with it. Use it once the car drives straight
  and you want the result to become the new center, and check the end stops
  afterwards.

Both count as unsaved changes until you press **Save**.

### Trimming from the transmitter

Map **Servo Trim Steering** on the [Adjustments](adjustments.md)
tab to trim from the transmitter. The Setup Wizard's **Trim and gain** step
sets this up for you:

- **Stepped** (trim buttons): each press moves the saved trim by one
  **Step**. A quick tap counts. Hold the button and it starts repeating
  after half a second. Only the first step of each press beeps. The trim is
  saved when you disarm.
- **Mapped** (a trim channel or knob): sets the live trim directly, within
  the adjustment's range. It is never saved, so the radio keeps the trim.

Trimming the steering moves every servo whose [Mixer](mixer.md) rule takes
its input from the steering (Stabilized Steering), front and rear. Each servo's
Reverse, its rule's weight sign and the Steering Invert are taken into
account. Servos fed by a raw RC channel, an override or a logic condition
aren't touched. When any servo on the steering reaches its trim limit, the
trim stops for all of them, so a pair of steering servos never get pulled
out of line.

Because a Mapped trim is never saved, a channel that is misread (for
example one that isn't valid yet just after power-up) can move the steering
by at most the trim limit and leaves nothing behind once the reading is
right again. The firmware also ignores a trim channel until the receiver
link has been there for a full second (at power-up and after a brief link
drop).

Stepped trims work on the bench too: the Trim column follows each press,
and **Save** keeps the result.

Keep the radio's own steering trim off the steering channel when the
controller trims. A trim left on the stick is read as steering input, which
the gyro holds against.

### Trim buttons on one channel

The radio's steering trim buttons can drive a Stepped trim on one spare
channel. The Setup Wizard's **Trim and gain** step sets this up (**Trim
buttons on one channel**), with Ethos as the example; on other radios
(EdgeTX, OpenTX and so on) set up the same mix and weights. By hand:

1. **Model → Trims**: leave the steering trim enabled, with **Audio** on so
   each press clicks.
2. **Model → Mixes**: keep the steering trim off the steering channel. An
   Ethos car model's Steering mix has no trim switch: add a **Free mix**
   with **Source** Steering and **Output** the steering channel (CH1), then
   delete the Steering mix.
3. Add a **Free mix** named TRIM: always on, **Source** Maximum,
   **Operation** Add, **Output** a spare channel. Add an action for each
   steering trim button, with the trim position as its active condition,
   setting the mix weight:

    | Trim button | Weight | Channel | Adjustment |
    |---|---|---|---|
    | T1 CW | 40% | about 1680-1730 µs | Servo Trim Steering, up |
    | T1 CCW | 30% | about 1630-1680 µs | Servo Trim Steering, down |

4. On the [Adjustments](adjustments.md) tab, add Servo Trim Steering as
   **Stepped**, always on, on that channel, with those two windows and a
   Step of 2 µs.

T1 is the steering trim on the radio the wizard was written against; use
your own steering trim buttons with the same weights. Each weight puts its
own value on the channel (100% is 2012 µs). With no button pressed, the
channel reads outside both windows.

!!! note "Upgrading from older firmware"
    Older firmware wrote Stepped trims straight into Mid. Anything trimmed
    that way stays in Mid after the upgrade, and the new saved trims start
    at 0. In the CLI, `servo trim` lists the saved trims and
    `servo trim <servo> <µs>` sets one; `diff` and `dump` include them.

## Bus Servos

For FrSky XACT servos on one F.Bus wire, see [FrSky Servos](../../setup/frsky-servos.md)
and the [XACT Servo Programming](xact-servo.md) tab.

Enabling **SBUS Output** or **FBUS Master** on a serial port (see
[Configuration](configuration.md)) sends the servos down that UART too, for
digital bus servos wired to it instead of individual PWM wires. Servo N is
channel N on the bus. It is the same servo as PWM output N, with the same
Center, Min/Max, Scale, Trim, Speed and Reverse, so one
[Mixer](mixer.md) rule and one row in the table drive both.

The table lists the servos in use: servo 1 up to the board's PWM output
count or the bus channel count (see
[Bus output channel count](#bus-output-channel-count)), whichever is
larger. A note above the table says which servos the bus sends and which
are also on a PWM output. On a board with four PWM outputs and a
12-channel bus, for example, servos 1-12 are listed, all twelve go down the
bus, and servos 1-4 are on the PWM outputs as well.

To send something on the bus that isn't on a PWM output, give it a servo
number the board has no PWM output for.

### Bus output channel count

How many channels each bus output sends is set with **F.Bus output channels**
and **SBUS output channels**, above the table. Each appears only when a
port has that output assigned. A change applies straight away, the table
resizes to match, and it is saved with the rest of this tab. The CLI settings are
`fbus_master_channels` and `sbus_out_channels`:

| Output | Channel counts | Default | Frame |
|---|---|---|---|
| F.Bus | 8, 12, 16, 24 | 24 | 8 uses the 8-channel frame, 12 and 16 the 16-channel frame, 24 the 24-channel frame |
| SBUS | 8, 12, 16 | 16 | Always the 16-channel SBUS frame |

Channels past the count are sent at center. Pick 16 or less for F.Bus if
anything on the bus only accepts the 16-channel frame.

SBUS and F.Bus output can run at the same time, each with its own count.
Servo N is channel N on both, and the table follows the larger of the two
counts.

With a count of 16, channels 17 and 18 are also sent as on/off channels,
driven by servos 17 and 18: on when that servo's output is at 1500 µs or
above. Both are listed in the table. The 24-channel F.Bus frame has no
on/off channels.

With a 24-channel F.Bus receiver, receiver channels 19-24 are available too
(see [Receiver](receiver.md#channel-assignment)), so they can be mixed
straight to servos 19-24.

!!! note "Upgrading from older firmware"
    Older firmware had separate PWM servos (S1-S8) and bus servos (S9 and
    up), with a **Clone PWM outputs to bus servos** switch, and a Rate for
    each servo. All of that is gone. Servo and mixer rule settings reset to
    their defaults on the upgrade. In a backup made before it, a line that
    names servo 9 or above meant a bus channel 8 lower than it does now:
    the Configurator leaves those lines out when it restores such a backup,
    and they must not be pasted back into the CLI as they are. Set those
    servos up again by hand.
