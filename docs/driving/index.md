# Driving Modes & Features

Gripora stabilises one thing: the **steering**. A yaw-rate gyro loop turns
the steering servo so the car rotates at the rate the steering wheel asks
for, and pushes back when the car starts to slide or is knocked off line.
The throttle goes to the ESC as you give it, shaped only by the optional
drive aids.

| Page | Summary |
|---|---|
| [Car Mixer and Channels](car-mixer.md) | The default car setup: steering on S1, a reversible ESC on M1, optional rear steering and second ESC, the car channel order (CH1 steering, CH2 throttle) and the gyro direction setting |
| [Drive State](drive-state.md) | How the controller works out which way the car is travelling and how fast, so the gyro corrects the right way in reverse and fades out near a standstill |
| [Drive Aids](drive-aids.md) | ABS and traction control: timed shaping of the brake and throttle you give, per PID profile |
| [GYRO OFF](gyro-off.md) | Turns the steering gyro off from a switch: the steering stick goes straight to the steering servo |
| [Ready-to-Arm Wiggle](ready-to-arm-wiggle.md) | The steering twitches to show the car is ready to arm, armed, or can't arm |
| [Backup RX Input](backup-rx-input.md) | A second or third receiver takes over if the main link is lost |

## Modes

Modes are put on transmitter switches on the
[Modes](../configurator/tabs/auxiliary.md) tab. The ones that change how the
car drives:

| Mode | What it does |
|---|---|
| **ARM** | Arms the ESC output. Arming needs the throttle at neutral. |
| **PREARM** | Optional safety switch that must be on before ARM works. |
| **GYRO OFF** | Steering without the gyro. See [GYRO OFF](gyro-off.md). |
| **DRIVE AIDS OFF** | ABS and traction control off; the gyro still steers. See [Drive Aids](drive-aids.md). |
| **FAILSAFE** | Tests or triggers failsafe from a switch. See [Failsafe](../configurator/tabs/failsafe.md). |

The other modes (beeper, LED, Blackbox, telemetry, camera control, user
outputs) are described on the [Modes](../configurator/tabs/auxiliary.md) tab
page.

## Throttle, brake and reverse

The throttle is signed around the centre of the throttle channel
(`rc_center`, 1500 µs by default): forward above it, brake and reverse below
it, with a small neutral deadband. "Throttle off" always means the trigger
at neutral, not at full brake. That is what arming, auto-disarm and failsafe
check for, and what failsafe sends to the ESC (`failsafe_throttle`, 1500 µs).

What the ESC does below neutral (brake, reverse, or brake then reverse) is
set in the ESC itself. Tell the controller which it is on the
[Motors](../configurator/tabs/motors.md) tab's Drive panel; see
[Drive State](drive-state.md).

!!! warning "Racing rules"
    Sensor-based driving aids are banned in sanctioned racing. The steering
    gyro is one; class rules may also ban ABS and traction control even
    though Gripora's versions use only the stick and time. Check the rules of
    your class and sanctioning body before running Gripora at a race.
