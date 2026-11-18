# Blackbox

The Blackbox tab configures data logging: which fields are recorded, the
logging rate, and (if fitted) the onboard flash or SD card. Blackbox logs
are the main tool for diagnosing tuning problems after a run, and the first
thing to attach when asking for tuning help.

## When it logs

**Device** picks where logs go: onboard flash, an SD card, or an external
serial logger (each only appears if fitted or supported). **Mode** then
decides when logging is active: **Armed** (simplest: logs every run),
**Normal** (logs only when armed *and* a BLACKBOX switch is on, to keep only
the runs you choose), or **Switch** (logs whenever that switch is on, armed
or not). **Grace Period**
keeps logging briefly after disarming, specifically to catch the moments
right after a crash rather than cutting off exactly at disarm.

## Rate and fields

**Rate of Logging** is a divisor of the PID loop rate -- higher rates
capture more detail (useful for chasing a specific fast oscillation) at
the cost of filling storage faster; lower rates log longer runs in the
same space. The field checkboxes choose which data categories are
recorded at all -- turn off what you don't need to save space, but when in
doubt for a tuning question, leave gyro, PID and setpoint data on, since
that's what most tuning advice is read from. The throttle (`rcCommand[4]`)
is signed: positive forward, negative brake and reverse.

**Debug Mode** adds a further 8 extra values on top of the normal fields,
for a specific diagnostic (its exact meaning depends on which mode is
selected) -- leave it on `NONE` unless you're chasing something specific
that calls for it. Two are specific to Gripora's car features: `DRIVE`
(the [drive state](../../driving/drive-state.md): throttle, ESC state,
direction, speed and gyro correction scale) and `DRIVE_AIDS` (the
[drive aids](../../driving/drive-aids.md): throttle in and out, ABS and
traction control). `GAIN_ATTEN` shows the throttle and speed gain
attenuation. Modes that no longer record anything are named `UNUSED_<n>`
and are left out of the list; see the
[CLI Command Reference](../../reference/cli-reference.md).

## Flash storage

**Initial Erase** guarantees a minimum amount of free space before
recording starts, erasing old logs if needed -- note that erasing can take
a while on some flash chips, so recording doesn't begin until it finishes.
**Rolling Erase** instead overwrites the oldest logs once storage is full,
trading guaranteed history for never running out of space -- logs can gain
gaps if the chip is slow to erase mid-run.

## Reading mode changes in logs

Each log frame records which modes are active in two words,
`flightModeFlags` and `flightModeFlags2`, together covering every mode. A
tool that only reads `flightModeFlags` misses the modes past the 32nd. The
separate mode change *event* carries only the first word.

The log header records the drive aids settings of the profile in use
(`drive_abs`, `drive_tc`).

## Log files over USB

When the controller is plugged in as a USB drive (mass storage), log
files are named after the craft name (Configuration tab), followed by
a sequence number and timestamp. With no craft name set, the prefix is
`grpa` (it was `wflt` in WingFlight and `rtfl` in Rotorflight), for example
`grpa_002_20251012_141213.bbl`.

## Reviewing logs

Open a `.bbl` file in the Gripora Blackbox viewer to play the run back.
Each graph stacks related fields, such as the steering setpoint against the
yaw gyro, then the servo and motor outputs, with the legend showing every
field's value at the cursor. The strip along the bottom shows the whole log
so you can jump to any part of the run. **Graph setup** chooses which fields
each graph shows.

The **Overlay** buttons add analysis views on top of the graphs. The
spectrum analyser plots the frequency content of a field over the visible
window, with the gyro filter cutoffs marked, to find the frequency of a
vibration before changing filter settings. The step response graph shows
how quickly and cleanly the gyro follows a change in setpoint; a trace that
overshoots well past 1.0 or keeps ringing points to gains that are too
high.
