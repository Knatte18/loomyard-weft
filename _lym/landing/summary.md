# Name a hub's desktop launcher "lym: <shortname>"

A hub's desktop launcher, written by `internal/desktoplauncher`, was named `lym: start the hub <shortname>`, which is too long to read under a desktop icon.
It is now named `lym: <shortname>`, such as `lym: odd`: the shortname alone tells the hubs apart.

The name shows in the `Name=` key of the Linux `.desktop` file and in the `rem` line of the Windows `.cmd` file.
The launcher's file names, its `Comment=` line and the command it runs are unchanged,
so a standing launcher is rewritten in place on the next dobby start that adds it.

The all-hubs launcher keeps `lym: start every hub`, since the request named only the per-hub launcher.

The package doc now names both displayed names,
and the rows of `TestAdd_WritesLaunchersRunningMachineStart` expect the new hub name on each form.

## Merge danger

**Door:** two-way
**Blast radius:** desktoplauncher
