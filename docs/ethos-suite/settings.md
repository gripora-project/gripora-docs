# Settings

**Settings** are the suite's own options. They are stored on the radio, not
on the controller, so this menu works with the car switched off. Saving
here asks before it writes, like every other page, but the settings go to
the radio.

![The Settings menu](../assets/images/ethos-suite/settings_menu.png)

## General

Grouped into panels; tap a panel to open it.

![Settings, General page](../assets/images/ethos-suite/settings_general.png)

| Panel | Settings |
| --- | --- |
| Display | How the radio's own battery is shown (*Tx Battery Options*), and the temperature unit. |
| Developer | *Developer mode*, which adds a Developer menu under **Tools**. You won't need it for driving. |
| Safety Prompts | *Save confirm* and *Reload confirm*: turn off to save or reload without being asked. |
| Integration | *Sync model name*: while the car is connected, the radio shows the name set on the controller as the model name, and puts the old name back afterwards. *Battery profile on connect*: asks which battery is fitted each time the car connects. |

## Dashboard

Choose and set up the [dashboard](dashboard.md) theme.

![Settings, Dashboard menu](../assets/images/ethos-suite/settings_dashboard_menu.png)

- **Themes:** the default theme for all models, and optionally a different
  theme for this model.
- **Settings:** one tile per theme, for the options that theme has.

![Settings, Dashboard, Themes page](../assets/images/ethos-suite/settings_dashboard_theme.png)

## ActiveLook

For ActiveLook glasses: what the glasses show before, during and after a
run (**Before Run**, **During Run** and **After Run**). **Settings** turns the glasses display on and sets its position; the
other three pages each pick a layout and what goes in each slot.

![Settings, ActiveLook menu](../assets/images/ethos-suite/settings_activelook_menu.png)

![Settings, ActiveLook, Before Run page](../assets/images/ethos-suite/settings_activelook_preflight.png)

## Audio

What the radio says, and when.

![Settings, Audio menu](../assets/images/ethos-suite/settings_audio_menu.png)

### Events

Callouts and alerts, one page per kind:

![Settings, Audio, Events menu](../assets/images/ethos-suite/settings_audio_events_menu.png)

| Page | What it covers |
| --- | --- |
| Voltage | Low battery voltage, and BEC and receiver voltage alerts, with their thresholds. |
| ESC temp | An alert when the ESC gets hotter than a threshold. |
| Fuel | Fuel (charge left) callouts, and the low-fuel warning. |
| State callouts | Arming flags, the mode, GPS fix, PID, rate and battery profile changes, and adjustments. |
| FC status | Alerts from the controller, such as the backup receiver taking over or the GPS not responding. |
| Link | An alert, and optionally a vibration, when telemetry is lost. |
| Model callout | Plays the car's name when it connects, if you have recorded it. |

![Settings, Audio, Events, State callouts page](../assets/images/ethos-suite/settings_audio_events_state.png)

See [Radio Callouts and Alerts](../reference/radio-alerts.md) for what each
alert means and what to do about it.

### Switches

Pick a switch for a value, and the radio reads that value aloud when you flip
it: voltages, consumption, current, ESC temperature, RPM and more.

![Settings, Audio, Switches page](../assets/images/ethos-suite/settings_audio_switches.png)

### Timer

Callouts for the run timer: when it runs out, and alerts before and after.

![Settings, Audio, Timer page](../assets/images/ethos-suite/settings_audio_timer.png)
