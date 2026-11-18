# Installing the Lua Suite

## 1. Download and copy to the radio

Use either way:

- **With the updater:** the
  [Gripora Radio Updater](https://github.com/gripora-project/gripora-lua-ethos-suite-updater/releases)
  for Windows, macOS and Linux downloads the suite and copies it to the
  radio.
- **By hand:** download the suite from the
  [releases page](https://github.com/gripora-project/gripora-lua-ethos-suite/releases).
  Each release has one zip per language, for example
  `gripora-lua-ethos-suite-<version>-en.zip` for English. Pick the release
  that matches your firmware version: the release notes say which firmware
  it is for. Then either:
    - **Ethos Suite:** connect the radio by USB, open *Lua Development
      Tools*, choose *Install Lua Scripts*, and select the zip, or
    - **as a drive:** connect the radio by USB as a drive, unzip the
      download, and copy the `grsuite` folder into the `scripts` folder on
      the radio. Replace the old `grsuite` folder if there is one.

Then unplug the radio and restart it.

## 2. Turn on the background task for your model

The suite does its work in a background task that each model has to turn on.
On the radio, open the **Model** menu, go to page 3, and open **Lua**. Turn
**Gripora [Background]** on.

![Model menu, Lua page, with Gripora [Background] turned on](../assets/images/ethos-suite/model-lua-task.png)

Until it is on, every tile in the app shows *Background task not running*
when you press it.

## 3. Check telemetry

The app talks to the controller over telemetry, so the radio needs a
working link to the receiver:

1. In the **Model** menu, **RF system**, make sure the RF module you drive
   with is turned on.
2. Power up the car, then open **Model** → **Telemetry** and discover
   sensors if the list is empty.

If sensors are missing, see [Telemetry](../reference/telemetry.md) for the
controller side.

## 4. Open the app

Press **SYS** on the radio and go to the second page. The **Gripora** tile
opens the app.

![System menu, page 2, with the Gripora tile](../assets/images/ethos-suite/sys-menu-tile.png)

With the car powered and connected, every tile in the app can be opened.
Without a connection, only **Tools** (for the radio-side diagnostics),
**Logs** and **Settings** work: everything else needs the controller.
Continue with [Using the app](using-the-app.md).

## Updating

Install a new release the same way. Your app settings (under **Settings**)
are kept in a separate folder, `scripts/grsuite.user`, so replacing the
`grsuite` folder does not lose them.
