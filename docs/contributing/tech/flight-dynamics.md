# Steering Control Dynamics: Signal Chain, Design Rationale and Known Defects

This document describes how Gripora turns the driver's steering and throttle into steering-servo and ESC
commands. It records why each stage is built the way it is, and lists the known defects and limitations. Read
it before changing steering-control code: `src/main/flight/{setpoint,pid,drive,drive_aids,speed_atten,mixer,servos,motors}.c`,
`src/main/fc/rc.c`, `src/main/fc/core.c` and `src/main/flight/failsafe.c`.

The page keeps its old address (`contributing/tech/flight-dynamics/`) because source comments and the firmware's
`AGENTS.md` link to it. Directory and file names such as `src/main/flight/` are inherited from the fixed-wing
firmware Gripora was forked from (WingFlight).

**Scope.** Everything between the receiver and the servo/motor outputs: RC processing, setpoint shaping, the
steering rate PID, the drive state, speed and throttle gain attenuation, GYRO OFF, the drive aids (ABS and
traction control), the mixer, servo and motor output, and the arming and failsafe logic that overrides them.

**Basis.** A static read of the firmware source at the time of writing, checked against the unit tests where
they exist. Nothing here was verified by driving. Where a statement depends on behaviour that can only be
observed on a car, it says so.

---

## 1. Conventions

These are easy to get wrong.

| Item | Convention | Where |
|---|---|---|
| Axes | Only yaw (steering) is stabilised. Roll and pitch keep their slots in every per-axis array, PID and rate profile, MSP message and Blackbox field, but their setpoints and PID outputs are always zero. | `setpointUpdate()` zeroes roll/pitch setpoints; `pidResetRollPitch()` in `pid.c` |
| Yaw sign | RC yaw (steering stick) is clockwise-positive; gyro yaw is negative for the same motion. `setpoint.c` negates the RC yaw **once**, and everything downstream keeps that sign. GYRO OFF negates the raw channel the same way, so the steering direction is the same with and without the gyro. | `setpointUpdate()`, `mixerUpdateInputs()` |
| Pitch sign | `attitude.values.pitch` / `attitude.raw[PITCH]` are **positive nose-down** (inherited Betaflight convention). Nothing in the steering loop uses pitch; it matters for attitude telemetry and the Blackbox `attitude` field. | `imu.c` |
| Throttle | Signed, -1..1 around `rc_center`: forward above, brake/reverse below. Inputs within 10 µs of centre are neutral (`RC_THROTTLE_NEUTRAL_DEADBAND`). "Throttle off" for arming, auto-disarm and failsafe is the stick within 15 µs of centre (`isThrottleOff()`: that deadband plus `RC_THROTTLE_DEADBAND`, 5 µs). | `fc/rc.c` |
| Channel order | Default map `RT1234AE`: CH1 steering, CH2 throttle, CH3-CH6 AUX1-4. Roll and pitch are unused and parked on CH7/CH8. | `pg/rx.c` (`rcmap`) |
| PID output | The PID sum is a unitless steering command: ±1.0 is the full stabilized-yaw mixer input, which the default rule sends to S1 at weight 1000, i.e. full servo scale. | `pidSum` into `MIXER_IN_STABILIZED_YAW`; limits ±1000 in `pg/mixer.c` |
| Units | Setpoint and gyro rate in °/s. `axisError` (the I accumulator) in degrees, limited by `error_limit` (degrees). Drive speeds in km/h (settings in 0.1 km/h). | `pid.c`, `drive.c` |
| Loop timing | `pid.dT` comes from `gyro.targetLooptime`. With `pid_process_denom` > 1, `taskMainPidLoop()` in `core.c` spreads setpoint → PID → mixer → servo across consecutive ticks. | `activePidLoopDenom` cases |

### Signal chain

```
 RX (rx.c) ─► rcInput ─► rc.c: deadbands, ±1 deflection; throttle signed -1..1 around rc_center
                              │
                              ▼
        setpoint.c: yaw sign ─► smoothing ─► yaw dynamic deadband/ceiling
                    ─► response/accel limit (sp.deflection) ─► boost ─► rates curve (sp.setpoint)
                    roll and pitch setpoints forced to 0
                              │
                              ▼
        pid.c: driveUpdate() + speedAttenUpdate(), yaw I cleared on a direction change
               yaw gyro × (yaw_gyro_reverse ? -1 : 1) × travel direction (±1)
               P, D: Kp/Kd · masterGain · gainCurve · atten · corr
               I:    Ki · masterGain · corr · axisError
               F, B: Kf/Kb · atten            (atten = max(TPA · SPA, 0.25), corr = drive correction scale)
                              │
                              ▼
        mixer.c inputs: STABILIZED_YAW = pidSum   (GYRO OFF: raw steering channel instead)
                        STABILIZED_THROTTLE = driveAidsApply(throttle)   (failsafe: failsafe_throttle instead)
                              │
                        rule table (default: STABILIZED_YAW → S1, STABILIZED_THROTTLE → M1)
                              │
              servos.c (curve, speed, reverse, scale, trim, travel)      motors.c (bidirectional: 0 = neutral)
```

`drive.c` reads the stabilized throttle the mixer sent on the previous loop (failsafe included) as its
command input, so the drive state follows what the ESC is actually told to do.

---

## 2. Subsystems: what they do and why

### 2.1 RC processing — `fc/rc.c`

Each stick is centred on `rc_center`, a per-axis deadband is removed (`roll_deadband`, `pitch_deadband`,
`yaw_deadband`; the throttle uses a fixed 10 µs), and the result is scaled to ±1 by `rc_deflection`. The throttle
is that same signed deflection: there is no 0..1 throttle any more. `getThrottle()` returns it; the
Blackbox logs it x1000 as `rcCommand[4]`. AUX channels are passed on as `rcInput − rc_center`.

### 2.2 Setpoint shaping — `flight/setpoint.c`

Order of operations: yaw negation, smoothing (cutoff derived from the measured RX frame interval), yaw dynamic
deadband/ceiling, response-time and acceleration limit (→ `sp.deflection`), setpoint boost, rates curve
(→ `sp.setpoint`). Roll and pitch are computed and then set to zero; their stick deflection is still available to
mixer rules through the RC inputs.

- **Smoothing tied to the RX frame rate.** Inherited from Rotorflight: it removes stair-stepping from slow link
  updates without a fixed lag.
- **Rates.** `rcRates × 5` °/s at full stick on the default curve. The default yaw rate is 150 °/s (`pg/rates.c`);
  the Configurator's setup wizard driving styles set 90 (Beginner), 150 (Sport) or 300 °/s (Drift).
- **Yaw dynamic deadband** is on by default (`yaw_dynamic_deadband_gain` 10), inherited.

### 2.3 Steering rate PID — `flight/pid.c`

`error = setpoint − gyroRate`, on yaw only (`pid_mode` 1; any value other than 0 runs mode 1). Before the error
is formed, the yaw gyro rate is negated if `yaw_gyro_reverse` is on, and multiplied by the travel direction from
the drive state (§2.4). Terms:

| Term | Formula | Notes |
|---|---|---|
| P | `Kp · masterGain · gainCurve · atten · corr · error` | |
| I | `Ki · masterGain · corr · axisError` | Not attenuated by TPA or SPA. Accumulation is also scaled by `corr`. |
| D | `Kd · masterGain · gainCurve · atten · corr · d/dt(−gyro)` | Gyro-only D (no setpoint kick). Default D = 0. |
| F | `Kf · atten · setpoint` | Carries the steady stick response. Not scaled by `corr`, so the stick always steers. |
| B | `Kb · atten · d/dt(setpoint)` | Feedforward boost. Not scaled by `corr`. |

`atten = max(TPA × SPA, 0.25)` (`PID_ATTENUATION_MIN`); `corr` is `driveGetCorrectionScale()` (§2.4).

**Default authority** (defaults from `pg/pid.c`, scale constants from `flight/pid.h`): yaw P 190, I 45, D 0,
F 65, B 35, `error_limit` 60°.

| Quantity | Value |
|---|---|
| P at 100 °/s error | 190 × 6.67e-6 × 100 ≈ 0.13 of full travel |
| I ceiling (Ki · `error_limit`) | 45 × 2e-4 × 60 = 0.54 of full travel |
| F per °/s | 65 × 2.5e-5 = 0.0016, so 615 °/s of setpoint is full travel from F alone |
| F at full stick, default 150 °/s | ≈ 0.24 of full travel |

**Design decisions:**

- **Gyro reversed for reverse travel, stick unchanged.** The yaw response to a steering angle is proportional to
  the signed speed (yaw rate ≈ v · tan(steer) / wheelbase), so reversing flips its sign and a forward tune
  becomes positive feedback. The stick keeps meaning "wheel angle" in both directions, so correcting in reverse
  is exactly negating the gyro rate the loop sees: P, I and D all follow, F and B are untouched (`pid.c`
  `pidApplyGyroRate()`, `drive.c` header comment).
- **`yaw_gyro_reverse`** (PID config PG v4) negates only this loop's yaw gyro, for when the gyro corrects the wrong
  way (typically a reversed steering channel fixed with a servo reverse). Board orientation and the attitude
  estimate are unchanged. It is in `MSP_MIXER_CONFIG` as an optional trailing byte.
- **Correction scale on P, I and D only.** At a standstill there is no yaw response to correct with, so P and D
  would only push the steering and I would wind up; F and B carry the stick and must always steer the wheels.
- **I-term cleared on a direction change**, because it was accumulated for the other sign of the plant.
- **I held at zero while disarmed** (yaw only): on the bench the car cannot respond, and whatever built up would
  arrive as a steering bump on arming. P and D still act while disarmed so the setup wizard's gyro-direction
  check (turn the car by hand, the wheels steer against it) works.
- **Throttle gain attenuation (`fw_tpa_gain`, `fw_tpa_curve`).** A baseline scale times an optional curve from the
  shared gain-curve pool, evaluated on |throttle| (0..1), mirroring `master_gain` + `gain_curve`. Applied to P,
  D, F and B, not I.
- **Speed attenuation (`fw_spa_gain`, `fw_spa_curve`, `fw_spa_speed_max`)**, `flight/speed_atten.c`. A car's yaw
  response to a steering angle grows with speed, so a tune that is right at moderate speed can oscillate flat
  out; TPA cannot see that when the throttle is closed at speed. Same shape as TPA, with the curve evaluated at
  `speed / fw_spa_speed_max` (default 60 km/h, range 10-300). Multiplied with TPA.
    - **Speed source:** the RPM wheel speed from the drive state when there is an RPM source, otherwise GPS
      (`max(speed3d, groundSpeed)`; only u-blox reports 3D speed).
    - **Filtering:** low-passed at 1 Hz (`FW_SPA_SPEED_CUTOFF_HZ`).
    - **Speed loss** (no RPM source and no GPS fix): hold the last scale for 3 s, then slew back to 1.0 at 0.2/s.
      Engaging (first speed, speed regained, profile change) slews onto the curve at the same rate.
    - With no curve SPA is exactly 1.0. Per PID profile, in `PG_FW_SPA_CONFIG` (v1). Unit-tested in
      `speed_atten_unittest`.
- **Master gain applied at the point of use**, not baked into `coef[]`, so live adjustments stay correct whichever
  adjustment last touched a coefficient.
- **I-term relax** (`iterm_relax`, the "bounceback" score 1-10, default 5, and `iterm_relax_level`, default
  22 °/s): a high-pass of the setpoint scales I accumulation down during fast steering input.
  `pidItermRelaxCutoff()` maps the score to the setpoint low-pass cutoff (50, 30, 20, 15, 10, 8, 7, 6, 5, 3 Hz).
  Always on.
- **I-term decay** (`iterm_decay_time`, default 60 = 0.6 s time constant, range 1-100; `iterm_decay_limit`,
  default 35 °/s): I bleeds continuously, so a stale I-term does not hold the steering off centre.
- **Anti-windup:** accumulation stops only when the stabilized-yaw mixer input is saturated *and* the error would
  push further into saturation (`pidAxisSaturated()`). Servo travel clipping also raises saturation
  (`mixerSaturateServoOutput()`), so trim-induced clipping is covered. Saturation flags last
  `MIXER_SATURATION_TIME` (5) ticks.
- **I held in GYRO OFF.** While GYRO OFF drives the steering (`mixerStabilizationBypassed()`, false once failsafe
  is active) the gyro is not tracking the setpoint, so I is not integrated; decay still runs.
- **Gyro overflow** (`gyroOverflowDetected()`): `pidReset()` zeroes all PID state and outputs.

**Mode 0** (`pid_mode` = 0) outputs F only: a stick-proportional command with no gyro.

### 2.4 Drive state — `flight/drive.c`, `PG_DRIVE_CONFIG`

The drive state estimates the travel direction and the wheel speed for the steering loop and the drive aids.

- **Inputs.** The stabilized throttle sent to the ESC (failsafe included), and the motor RPM when motor 1 has an
  RPM source (DShot telemetry, ESC telemetry or a frequency sensor), averaged with motor 2 if it has one too.
  RPM is a magnitude: no source reports a direction, so the direction comes from what the ESC was commanded to
  do. Wheel speed = motor RPM ÷ `motor1_gear_ratio` × π × `drive_wheel_diameter`, filtered at 10 Hz. Stopped =
  below 1 km/h for 0.1 s.
- **ESC model, `drive_esc_mode`.** The commanded state (neutral, forward, brake, reverse) follows the ESC's own
  rules. A command within 5 % of neutral is neutral.
    - `FWD_BRK_REV`: below neutral brakes after forward; back to neutral (held at least 50 ms,
      `DRIVE_NEUTRAL_DWELL_S`), then below again, reverses. A brake or reverse carries on while the stick stays
      below neutral. Reverse is also the first below-neutral command after arming.
    - `FWD_REV`: below neutral reverses, braking or coasting first while RPM shows the car still rolling forward.
    - `FWD_BRK`: below neutral only ever brakes.
    - `AUTO`: `FWD_REV` on a DShot protocol (the AM32 default), `FWD_BRK_REV` otherwise. An AM32 with "RC car
      reverse" on needs `FWD_BRK_REV` set by hand.
- **Direction with RPM** changes only at zero speed: an RPM-confirmed standstill, or the moment the wheels pass
  through zero (the unfiltered speed shows it). Braking while rolling forward is never taken for reverse. Wheels
  speeding up from zero under a below-neutral command are taken as reverse in every mode that can reverse, and
  under an above-neutral command as forward, so a reversal the model did not expect (a sensored ESC, an AM32
  reversing at its own threshold) is still caught.
- **Direction without RPM** follows the command at once (forward +1, reverse −1; brake and neutral keep it). The
  ESC may brake or coast for a while before it really changes direction, so on each change the correction scale
  drops to zero and fades back in over 1 s (`DRIVE_DIR_CHANGE_RAMP_S`).
- **Correction scale** = speed factor × reverse factor × ramp factor:
    - speed factor (RPM only): 0 below `drive_gyro_min_speed` (default 2.0 km/h, never below 1.0 km/h, the stop
      speed, so the correction is faded out before the direction can flip), rising linearly to 1 at
      `drive_gyro_full_speed` (default 6.0 km/h, always above the minimum). Without RPM it is 1.
    - reverse factor: 1 forward; in reverse `drive_gyro_reverse_gain` % (default 50), or 0 with
      `drive_gyro_reverse` OFF.
    - ramp factor: the 1 s fade after an unconfirmed direction change.
- **Disarmed** the state is held at neutral and forward, the ramp is complete and the speed factor is not applied,
  so the gyro corrects on the bench (with I held at zero, §2.3). Every arming starts from neutral and forward.
- **Debug** mode `DRIVE`: throttle command x1000, commanded state, direction, speed x10, speed known, correction
  scale x1000, motor 1 RPM, stopped. Unit-tested in `drive_unittest`.

### 2.5 GYRO OFF — `flight/mixer.c`

`BOXPASSTHROUGH`, shown as **GYRO OFF** (permanent id 12). While it is on and failsafe is not active, the
stabilized-yaw mixer input is replaced by the steering channel's deflection, negated to keep the stabilized sign
(§1): no rates, no expo, no PID. It is written directly (`mixerSetBypassInput()`), so the stabilized-yaw input's
min/max limits do not apply; the input rate and the servo's own limits still do. While disarmed, a bench override
or a wiggle still wins over the switch.

Only the steering is bypassed. The throttle still goes through the drive aids (§2.6); DRIVE AIDS OFF is the
separate switch for those.

### 2.6 Drive aids (ABS and traction control) — `flight/drive_aids.c`, `PG_DRIVE_AIDS_CONFIG`

The driver's throttle/brake command passes through `driveAidsApply()` before it becomes
`STABILIZED_THROTTLE`. An aid only ever moves the command towards neutral, never past it and never into the
opposite direction. Failsafe replaces the result afterwards, so a failsafe throttle is never shaped. The aids pass
the command through unchanged while disarmed, in failsafe and with **DRIVE AIDS OFF** (permanent id 66). A mixer
rule fed from the RC throttle input instead of `STABILIZED_THROTTLE` bypasses them. Every aid is OFF by default.
The settings are per PID profile (copied and reset with the profile).

- **ABS (timed).** Runs only while the drive state has the ESC braking (state BRAKE) and the car is not
  RPM-confirmed stopped, and the brake is past `drive_abs_trigger`. Each cycle (1 / `drive_abs_freq`) holds the
  brake command for half the cycle plus `drive_abs_hold` (at most 85 % of the cycle), then eases it to
  `drive_abs_floor` % of the command. `DYNAMIC` moves the rate towards `drive_abs_dyn_freq`, the floor towards
  60 % of `drive_abs_floor` and the hold towards `drive_abs_dyn_hold` as the brake deepens from the trigger to
  full. `drive_abs_corner_freq` adds rate and `drive_abs_corner_hold` moves the hold with steering-stick
  deflection; where both a dynamic and a steering hold apply, the shorter wins. The rate is limited to 2-33 Hz.
- **10 % minimum release.** A brake-then-reverse ESC that sees neutral in the middle of a brake reverses on the
  next pull, so the released brake never drops below 10 % brake (`DRIVE_ABS_MIN_BRAKE`). The command stays on
  the brake side of neutral, and the drive state stays BRAKE with it.
- **Traction control (`RAMP`).** A forward command may only rise at a limited rate, zero to full in
  `drive_tc_ramp_time` (50-2000 ms, default 500); letting off is instant. It shapes only a forward command, never
  a brake (including a command above neutral while the ESC brakes a car rolling backwards) or reverse. While
  it, or the aids, are off the limit follows the command, so turning it on with the trigger held never cuts the
  throttle.
- **Timing only.** Both use the sticks and time, no wheel-slip sensing.
- **Debug** mode `DRIVE_AIDS`: throttle in and out x1000, flags, ABS rate x10 (or TC limit x1000 when driving
  forward with TC on), floor x1000, hold ms, cycle phase ms, steering x1000. Unit-tested in `drive_aids_unittest`.

### 2.7 Mixer — `flight/mixer.c`, `pg/mixer.c`

A rule table: `output (SET/ADD/MUL) = (offset + weight · curve(input · rate)) / 1000`, with a separate negative
weight, an optional slew (`speed`) and an optional logic-condition gate. `model_type` is `CAR` (0) or `CUSTOM`
(1). The default (car) rules are `SET STABILIZED_YAW → S1` (steering) and `SET STABILIZED_THROTTLE → M1`
(reversible ESC); rear steering and a second ESC are added by the Configurator's car preset as further rules.
The stabilized throttle input range is −1000..1000.

- **Saturation flags** last 5 ticks and gate the PID's anti-windup.
- **Overrides and wiggle** apply only while disarmed. The wiggle (ready, armed, arming-error patterns,
  `wiggle_*` settings) moves only the steering (yaw) input (`flight/wiggle.c`).
- **Non-finite guard.** A NaN or infinity at a mixer input, a rule output or an accumulated output is replaced by
  zero (and counted as saturation at the input).
- **Live-adjustable rule weights** keep the configured polarity in `mixerRuleSign[]`, because a live magnitude
  passes through 0, which has no sign.

### 2.8 Servo output — `flight/servos.c`

Order: mixer output (or a bench override) → geometry correction → balance curve → non-finite guard → speed limit
(each servo on its own) → reverse → `rpos`/`rneg` scale → trim (saved + runtime) → travel limit → `mid`.

- There are 24 servos; servo N goes to PWM output N where the board has one and to channel N of an SBUS/F.Bus
  output, from one output stage. Every servo has the signal range 880-2159 µs and one PWM update rate
  (`servo_pwm_rate`).
- **Trim never edits `mid`**, so min/max (offsets from `mid`) stay where they are. Saved and runtime trim are
  added before the travel limit, so the output stays inside min/max.

### 2.9 Motor output — `flight/motors.c`, `drivers/`

Every motor defaults to `motor_control_mode` bidirectional, with `min_throttle` 1000 and `max_throttle` 2000. A
zero command to a bidirectional motor is neutral: on PWM, SRXL2 and SITL the midpoint of
`min_throttle`..`max_throttle`, never `min_command`, which a car ESC reads as full brake/reverse and many learn as
their neutral at power-up. On DShot zero is the motor-stop command, positive commands use the forward range and
negative ones the reverse range.

### 2.10 Attitude estimator — `flight/imu.c`

The inherited Mahony quaternion filter still runs on all three axes (gyro integration corrected by the
accelerometer, plus magnetometer or GPS course when available). Accelerometer readings are only used within
0.9-1.1 g; the gain is x10 while disarmed, with a quick re-convergence after a disarm. Nothing in the steering
loop or the drive aids uses the attitude: it feeds the arming check (§2.11), telemetry and the Blackbox.

### 2.11 Arming, running latch, failsafe — `fc/core.c`, `flight/failsafe.c`, `rx/rx.c`

- **Arming** is blocked unless the throttle is at neutral (`ARMING_DISABLED_THROTTLE`) and the attitude estimate
  is established (`isAttitudeEstimateReady()`, which does not check that the car is level), among the other
  arming-disable flags. While armed with the throttle at neutral the armed beeper sounds; with stick arming,
  `auto_disarm_delay` (default 5 s) at neutral disarms.
- **Running latch** (`INFLIGHT_MODE`, the telemetry RUNNING bit): set once armed and |throttle| > 0.1, held
  through stops, cleared on disarm. There is no airborne detection any more. A disarm while running, after at
  least `rearm_min_armed_seconds` (5 s) armed, opens a `rearm_grace_seconds` (10 s) window in which re-arming
  skips the throttle and attitude checks, so an accidental disarm can be recovered while the car is moving.
- **Receiver signal loss** (`rx.c`): bad channels hold their last value for 300 ms, then each channel takes its
  `rxfail` value. The default (AUTO) puts roll, pitch and steering at centre and the throttle at neutral
  (`getFailsafeThrottle()` returns the throttle centre); other channels hold their last value.
- **Failsafe** (`failsafe.c`) is monitoring from start-up and runs its state machine:
    - Armed with the throttle at neutral for at least `failsafe_throttle_low_delay` (10 s) before the link drops:
      disarm straight away ("just disarm").
    - Otherwise, after `failsafe_delay` (1.5 s) without a valid signal, `failsafe_procedure` runs: `DROP`
      (default) disarms at once; `AUTO-LAND` holds `failsafe_throttle` (default 1500 µs, neutral) for
      `failsafe_off_delay` (1 s), then disarms.
    - While a procedure is active, `failsafe_throttle` replaces the drive aids' output on the stabilized throttle,
      and GYRO OFF is ignored. Re-arming needs `failsafe_recovery_delay` (1 s) of valid signal.
    - A FAILSAFE mode switch acts as stage 1 (default), `KILL` (disarm at once) or `STAGE2`
      (`failsafe_switch_mode`).

---

## 3. Decision log (Gripora)

Gripora was forked from WingFlight 0.0.43 in November 2026. The fixed-wing history (attitude hold, thrust
vectoring, GPS navigation, the governor and so on) is in the WingFlight repository; none of that code remains.

| Commit | Decision | Why |
|---|---|---|
| `6a4250e76` | Remove the aircraft features (attitude hold, trainer, angle/leveling, GPS nav and rescue, airborne detection, auto trim, trim monitor, thrust vectoring, hold engine, governor, tune advisor); stabilise yaw only; signed throttle around `rc_center`; bidirectional neutral; ids kept reserved | Car conversion, step 1; reserved ids keep MSP, BOX and saved-profile layouts valid |
| `24ab3af49` | Car mixer preset (S1 steering, M1 ESC), steering-only wiggle, `yaw_gyro_reverse` | A gyro that corrects the wrong way must be fixable without reversing the driver's steering |
| `6a94592d0` | Drive state: gyro negated in reverse, correction faded with speed and reduced in reverse | Reversing turned the correction into positive feedback; at a standstill I wound up and slammed the steering |
| `4dff3f73b` | Default channel order CH1 steering, CH2 throttle | Car radios |
| `36fdb2256` | Gyro corrects on the bench (no speed fade disarmed); yaw I held at zero disarmed | The setup wizard's gyro-direction check needs a correction with the car still; no I bump on arming |
| `f67886c8f` | MSP API 33.0; `MSP2_GRIPORA_*` names | Gripora's protocol is its own from here |
| `d5e28efef` | Timed ABS, per PID profile; DRIVE AIDS OFF mode | First drive aid; a profile switch changes it with the tune |
| `259d57c55` | `fw_spa_speed_max` default 60 km/h, range 10-300 | Car speeds |
| `8afcd9c4d` | Timed traction control (ramp) | Second drive aid |

---

## 4. Known defects and limitations

**Confirmed** = follows directly from the code. **Needs verification** = depends on behaviour that can only be
observed on a car.

### High

**H-1. Without an RPM source there is no low-speed fade.** *Confirmed.* The speed factor is 1 without RPM
(`driveSpeedFactor()`), so armed at a standstill the full P and I correction runs. Holding steering off centre on
a stopped car then makes P push and I wind up towards full lock, because the car cannot yaw at the commanded rate.
The Configurator says so on the Motors tab. Fix: an RPM source (DShot telemetry, ESC telemetry or a frequency
sensor).

**H-2. `FWD_REV` with no RPM: braking is taken for reverse, and ABS never runs.** *Confirmed.* Without RPM,
`driveNextState()` cannot know the car is still rolling, so every below-neutral command is REVERSE. The travel
direction flips at once: the gyro rate is negated during what is physically still a forward brake, softened only by
the 1 s correction fade-in, and the yaw I-term is cleared. ABS needs state BRAKE, so it never runs. The Configurator
warns on the drive aids panel. Fixes: an RPM source, or `FWD_BRK_REV` if the ESC supports it.

### Medium

**M-1. The ESC model can disagree with the ESC.** *Needs verification.* Without RPM the direction is only what the
model predicts. `FWD_BRK_REV` reverses after 50 ms at neutral (`DRIVE_NEUTRAL_DWELL_S`); a real ESC may need a
longer or shorter neutral, or have its own reverse threshold. With RPM, a reversal the model missed is caught when
the wheels speed up from zero; without RPM it is not.

**M-2. Default steering authority from F is small.** *Confirmed arithmetic; needs verification on a car.* With the
defaults (yaw F 65, 150 °/s at full stick) feedforward gives about 24 % of full travel at full stick. With an RPM
source and the car below `drive_gyro_min_speed`, P, I and D are scaled to zero, so F and the transient B are all the
steering there is: an armed, stopped car may not reach full lock until it moves. Whether this matters in practice,
and whether F or the default rate should change, has not been checked on a car.

**M-3. Wheel speed is driven-wheel speed.** *Inferred, not verified.* Speed comes from motor RPM through the gear
ratio, so it reads the driven wheels. Wheelspin or locked wheels under braking make it differ from ground speed,
which affects the low-speed gyro fade, the "stopped" detection (which gates ABS) and speed attenuation.

**M-4. Drive aids act on the stick, not on the wheels.** *Confirmed (by design).* ABS and traction control are timed
from the stick and the clock. The ABS "steering" adjustments use the steering-stick deflection, not the actual
wheel angle. There is no slip detection; sensor-based modes are planned as separate modes.

### Low

- **L-1. GYRO OFF bypasses the stabilized-yaw input limits.** *Confirmed, intentional.* `mixerSetBypassInput()`
  writes the input directly, so a tightened `mixer input` min/max on stabilized yaw does not apply in GYRO OFF.
  Servo min/max still do.
- **L-2. GYRO OFF does not bypass the drive aids.** *Confirmed.* Only the steering is raw; ABS and traction control
  still shape the throttle unless DRIVE AIDS OFF is on. User-facing text that says the throttle "passes straight"
  in GYRO OFF is describing the steering only.
- **L-3. Arming does not check that the car is level.** *Confirmed.* `isAttitudeEstimateReady()` only checks that
  the attitude estimate is established. `ARMING_DISABLED_ANGLE` keeps its name because it is user-visible.
- **L-4. First IMU update integrates over a huge `dt`.** *Confirmed.* `previousIMUUpdateTime` starts at 0, so the
  first step is long. Inherited from Betaflight; converges quickly, happens once per boot.
- **L-5. Accelerometer accepted at 0.9-1.1 g with no correction for cornering or acceleration.** *Confirmed.* Only
  attitude telemetry and the arming check depend on it.

### Fixed in the inherited code and still in place

- **Failsafe state machine runs.** The inherited Rotorflight code had `failsafeStartMonitoring()` commented out,
  so failsafe never ran; WingFlight re-enabled it, and it is called from `core.c`. (Older notes that say "stage 2
  is disabled" are out of date.) There is still no running automated test (`flight_failsafe_unittest.cc.txt`).
- **Non-finite guard** at the mixer inputs, rule outputs and the servo boundary.
- **Each servo is speed-limited on its own** (the inherited helicopter code scaled all cyclic servos together).

### Not reviewed

Dynamic notch and RPM filters, gyro/accelerometer drivers and calibration, `logic_condition.c`, Blackbox, MSP and
CLI plumbing.

---

## 5. Test coverage

Running unit tests (`make test`, `src/test/unit`) cover the PID (`pid_unittest`), setpoint, curves, maths, speed
attenuation, the drive state (`drive_unittest`), the drive aids (`drive_aids_unittest`), servos and telemetry
status. The mixer, IMU, failsafe and arming-prevention tests exist but are **disabled**
(`flight_mixer_unittest.cc.txt`, `flight_imu_unittest.cc.txt`, `flight_failsafe_unittest.cc.txt`,
`arming_prevention_unittest.cc.txt`), so the mixer, failsafe and `fc/core.c` arming paths have no running tests.

Worth adding, cheapest first:

1. **Drive state with the real ESC** (M-1): Blackbox the `DRIVE` debug mode while braking, reversing and
   re-reversing with and without RPM, and compare the commanded state and direction with what the car did.
2. **Standstill steering** (H-1, M-2): armed, stopped, full steering, with and without RPM; record
   `mixer[2]` and the servo output.
3. **Failsafe**: revive `flight_failsafe_unittest` for the just-disarm, DROP and AUTO-LAND paths with a signed
   neutral throttle.
4. **Mixer**: revive `flight_mixer_unittest` for GYRO OFF, the drive aids hand-off and the failsafe throttle
   replacement.

Nothing in this document has been verified by driving.
