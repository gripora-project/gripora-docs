# Ready-to-Arm Wiggle

The wiggle moves the steering to tell you what the controller is doing, so
you can see it without a beeper or LEDs in view. On a car only the steering
moves.

| Wiggle | When | What the steering does | CLI | Default |
|---|---|---|---|---|
| **Ready** | Nothing is blocking arming any more | One full left-right sweep, a quick twitch each way, then eases back to centre | `wiggle_enable_ready` | `ON` |
| **Armed** | You arm | A quick left-centre-right twitch (about 0.15 s); arming waits for it to finish | `wiggle_enable_armed` | `OFF` |
| **Error** | The ARM switch is on but arming is blocked | Shakes for 0.75 s, every 3 s | `wiggle_enable_error` | `OFF` |
| **Fatal** | The ARM switch is on and arming is blocked by something that needs fixing on the bench: no gyro, CPU load, RPM filter, a reboot required, accelerometer calibration, motor protocol, DShot bitbang | Pulsed shaking for 2.5 s, every 5 s | `wiggle_enable_fatal` | `OFF` |

All of them happen only until the first arm after power-up. The error
wiggles don't start during the boot grace time or while the Configurator or
CLI is connected. The Status tab, or CLI `status`, names the flags that block
arming.

Two settings shape the movement:

| Setting | Range | Default | Meaning |
|---|---|---|---|
| `wiggle_strength` | 0-100 | 50 | How far the steering moves, % of full travel |
| `wiggle_frequency` | 2-50 | 10 | Shake rate of the error and fatal wiggles, Hz |

A wiggle drives the steering while the car is disarmed, so keep hands clear
of the steering linkage when you power up.

Source: `src/main/flight/wiggle.c` and `src/main/fc/core.c` in the
firmware.
