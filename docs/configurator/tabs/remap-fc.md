# Remap FC

The Remap FC tab changes what each pin of the board does: a motor output, a
servo output, a serial port, the LED strip, the frequency (RPM) input and
so on. Add another ESC output or an extra servo, or move a function to a
free pin, without a soldering iron.

## Using it

1. **Read FC** reads the board's pins and their current use through the CLI.
   The tab shows the MCU and the board design it found.
2. Click a pin to see what it does, and choose what it should do. Picking a
   function moves it to this pin from wherever it was; picking the pin's own
   default puts it back. A pin that is hard-wired can't be remapped.
3. The tab suggests fixes for conflicts, for example swapping two functions
   or moving one to a free pin. Changes can't be sent while two functions
   share a pin, or while the motor or servo numbering has a gap.
4. **Save and Reboot** sends the changes as CLI `resource`, `timer` and
   `dma` commands and saves them.

Where the board has a configuration in
[gripora-targets](https://github.com/gripora-project/gripora-targets), the
tab uses its pin defaults.

The car setup on the [Mixer](mixer.md) tab tells you when the board has too
few servo outputs for rear steering; this is the tab to add one.

Take a [backup](../../getting-started/backup-and-restore.md) first. A pin
change is saved with the rest of the configuration (`diff all` includes
the `resource` lines).
