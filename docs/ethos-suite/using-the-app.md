# Using the App

## Menus

The app opens on its main menu. Every menu is a grid of tiles, sorted under
headings. A tile either opens another menu or a settings page.

![The main menu: Flight Tuning and Setup under Configuration; Tools, Logs and Settings under System](../assets/images/ethos-suite/root-menu.png)

You can use the touchscreen, or the rotary encoder: turn it to move between
tiles and press it to open one. When you go back to a menu, the tile you last
opened is still selected.

Tiles stay greyed out until the radio is talking to the controller.
**Tools**, **Logs** and **Settings** are the exception: they are stored on
the radio or check the radio side, so they open with the car switched off
(pages under Tools that need the controller still wait for it).

## The header

Every screen has a header row: the page's title on the left and its buttons
on the right.

![The PIDs page. The header shows the title PIDs #1 and the Back, Save, Reload and Tool buttons](../assets/images/ethos-suite/pids.png)

| Button | What it does |
| --- | --- |
| ![Back](../assets/images/ethos-suite/icon-back.png) **Back** | Goes back to the previous menu. RTN on the radio does the same. |
| ![Save](../assets/images/ethos-suite/icon-save.png) **Save** | Writes the page to the controller. |
| ![Reload](../assets/images/ethos-suite/icon-reload.png) **Reload** | Reads the page from the controller again, discarding changes you have not saved. |
| ![Tool](../assets/images/ethos-suite/icon-tool.png) **Tool** | The page's own extra action, where it has one (see below). |

Menus show only the **Back** button. On a settings page all four are there,
but a button is greyed out when it can't be used. For example, **Tool** is
grey on pages that have no extra action.

Pages that belong to a profile show its number in the title, such as
*PIDs #1* or *ABS #1*. The number follows the profile the controller is
using, so if you switch profiles from the radio, the page follows.

A long title loses its leading parts to fit, for example
*... / ESC temp* instead of *Settings / Audio / ESC temp* on a small screen.

## Changing and saving settings

1. Open a page. The suite reads it from the controller; a *Loading* dialog
   shows while it does.
2. Change a value: tap it, or select it with the encoder and press, then
   turn.
3. Press **Save**. It only lights up once you have changed something.
4. Confirm the save prompt.

![Save is lit after a change, and asks before writing](../assets/images/ethos-suite/save-dialog.png)

The controller keeps the change for good once it is saved. If you would
rather not be asked each time, turn the save and reload prompts off under
[Settings → General → Safety Prompts](settings.md#general).

!!! warning "Saving while armed"
    **Save** is greyed out while the car is armed. If the controller refuses
    a save because the car armed while it was being sent, a red banner says
    *Save not committed*: your changes are kept and written as soon as you
    disarm.

Use **Reload** to throw away changes you haven't saved. It asks before it
reads the page again.

## The Tool button

These pages have an extra action behind **Tool**:

| Page | Tool does |
| --- | --- |
| Accelerometer | Calibrates the accelerometer. Keep the car level and still. |
| Alignment | Turns the 3D view so the rear of the car faces you. |
| Telemetry | Selects the default telemetry sensors. |
| Servos | Turns servo override on or off. While it is on, a servo's centre moves as you change it, so you can trim the steering by eye. |
| Blackbox Status | Erases the onboard log memory. |

Each one asks before it does anything.

## Pages that don't work with Gripora

The suite came from WingFlight with pages for aircraft features that the
Gripora firmware no longer has, and they are being taken out of it. If your
version of the suite still shows any of these tiles, leave them alone: they
don't apply to a car, and the firmware refuses most of their requests.

- **Flight Tuning → Tune Advisor**
- **Flight Tuning → Flight Modes** (Acro Trainer, Angle Mode, Att Hold)
- **Flight Tuning → Thrust Vector** and its pages
- **Setup → GPS Navigation**
- **Setup → Governor**
