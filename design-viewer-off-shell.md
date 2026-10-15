# Run window stays on shell: follow the overview and newest agent from run start

`viewer-off-shell` · Note · bug, reed, batten

## Seen

On 2026-10-15 the operator's run window for the darn run `seed-ide-interactive-chain` sat on the `shell` window for the whole run.
It never moved to the overview (`loom-status`) or to the driver and `darn` agent windows as they spawned.
The same complaint has come up before: window auto-switching during runs is still unreliable.

## Why, by the documented rules

The focus rules in `internal/reedengine/doc.go` say that no automatic switch moves a viewer off the shell.
Batten's Open-Window row attaches the Konsole window right after Seed-Child, before the run has spawned its overview or any agent.
Attach selects `relevantWindow`, which is none at that moment, so the viewer stays on `shell`.
After that, the overview window is never selected, and a new agent window is selected only from the overview, a dead agent window, the relevant window or a driver window.
A viewer that landed on the shell by default is treated like one the operator moved there by hand, so it never follows the run.
A second possible cause: VS Code's folderOpen task chain also runs `lyx reed attach` (issue #633, task `seed-ide-interactive-chain`), and attach appends no `select-window` when another client is already attached.

## Wanted (operator)

When a run starts, the operator's window goes to the overview.
As the run spawns agents, it moves to the newest agent.
The shell is never where the window rests when a run starts.

## Idea

Tell "on the shell because nothing else existed at attach" apart from "moved to the shell by hand".
For example, an attach that found no relevant window marks the viewer as not placed, and the first overview or agent window to open selects it.
