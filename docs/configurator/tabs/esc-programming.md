# ESC Programming

The ESC Programming tab reads and writes the settings of a supported ESC
through the controller: AM32, BLHeli_S and Bluejay, and ESCs from
Hobbywing (V5), Scorpion, YGE, OMP, ZTW, XDFly and FlyRotor, over the
controller's USB connection, without a separate ESC programming card.

The flow is: pick your ESC's manufacturer/firmware, select which ESC to
target if more than one is connected, wait for detection, then edit its
parameters directly -- **Change ESC** at any point restarts this from the
beginning if you picked wrong or want to program a different ESC.

Programming an ESC can make the motor turn unexpectedly, so the tab asks you
to confirm the wheels are off the ground before it starts. The car must be
disarmed throughout; programming is blocked while armed. An ESC with no
remote link needs a power cycle after saving for the changes to take
effect.

On an AM32 ESC, the reverse setting decides which **ESC behaviour** the
controller needs on the [Motors](motors.md#drive) tab: with RC car reverse
turned on, set **Brake, then reverse**.
