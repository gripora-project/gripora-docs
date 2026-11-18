# Dashboard

The suite includes a dashboard widget for the model's home screen. It shows
the state of the car at a glance, and changes its layout for before,
during and after a run.

![The dashboard on the home screen](../assets/images/ethos-suite/home-dashboard.png)

## Adding it to a screen

The widget is called **Gripora Dashboard**. On the radio, press **DISP**
to configure the model's screens, pick a full-screen layout for the screen
you want, and choose **Gripora Dashboard** as its widget.

## Themes

A theme decides what the dashboard shows and how. Each theme has three
layouts: one before you arm, one while you drive, and one after you
disarm, and the dashboard switches between them by itself. The suite still
calls them *preflight*, *inflight* and *postflight*.

Choose the theme under [Settings → Dashboard](settings.md#dashboard): a
default for all models, and optionally a different one for a particular
model. Some themes have options of their own, under **Settings** on the same
menu.

[Dashboard Themes](dashboard-themes.md) shows each theme's three layouts.

## Toolbar

Swipe up on the dashboard, or long-press **PAGE**, to open the toolbar. It
closes by itself after 10 seconds, or swipe down.

![The dashboard toolbar](../assets/images/ethos-suite/dashboard-toolbar.png)

| Button | What it does |
| --- | --- |
| Reset | Resets the run's timers and minimum and maximum values, after asking. |
| Erase | Erases the controller's blackbox log memory, after asking. Greyed out until the car is connected. |
| Battery | Picks the battery profile. Greyed out until the car is connected, or if fewer than two battery profiles are set up. |
| Info | Opens the info panel (below). |
| Setup | Opens the Gripora app. Needs Ethos 26.1 or later. |

## Info panel

Swipe down on the dashboard, or pick **Info** on the toolbar, for a summary
of the car's state. It stays open until you swipe up, tap the dashboard,
press **RTN** or **ENTER**, or arm the car.

![The dashboard info panel](../assets/images/ethos-suite/dashboard-info.png)

| | Shows |
| --- | --- |
| **Controller** | The telemetry link and its quality, the active mode, whether the car is ready to arm (and if not, why), the active profiles, BEC voltage and how full the blackbox is. |
| **Battery** | The active battery profile's cells and capacity, the pack voltage and the mAh used. |
| **GPS** | Only when the car has a GPS: the fix (*No fix*, *Fix* or *Home set*) and the satellite count. |

A row appears once its value is known. With no car connected, both columns
say *Not connected*.

The dashboard's alert banner and the spoken callouts are covered in
[Radio Callouts and Alerts](../reference/radio-alerts.md).
