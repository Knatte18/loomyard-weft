# A run resumed on a second machine opens no terminal window on it

`open-window-per-machine` · Note · bug, batten

Imported from [issue #742](https://github.com/Knatte18/loomyard/issues/742).

Seen when two batten runs started on another PC were adopted with `lyx fabric add <slug>` and resumed with `lyx batten run <slug> --window` on this machine.

Batten resumed both at `Run-Shed`.
`Open-Window` was already recorded done from the first machine, so no terminal window opened on this machine (the help says that a run resumed past that row opens no window).
The operator watches each run in its own terminal window, so they had no window for either run here.

Workaround: attach by hand with `cd <pair> && lyx reed attach`.

Expected: Open-Window counts as done per machine, so a run resumed on a machine that never opened its window opens one there.