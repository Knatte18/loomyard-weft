# Run window stays on shell: follow the overview and newest agent from run start

`viewer-off-shell` · Task · bug, reed, batten · running · Webster-Bouncer

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

## Operator decision (2026-10-15): a follow toggle

A per-session follow state, shown as a clickable button on the reed status bar:

- Follow is on when a run starts: the viewer goes to the overview, then to each newest agent window, from the shell too.
- Any window change the operator makes by hand (a status-bar click, Alt+Up/Down) turns follow off, and the button shows it off.
  Reed owns those bindings, so it can tell a manual switch from its own.
  A viewer the operator moved deliberately, for example to the shell, is then never disturbed.
- Clicking the button turns follow back on and jumps to the newest agent window, else the overview.

The button is one entry in the configurable button set of note `reed-status-buttons`.

## Orch call (2026-10-15), not an operator decision

The operator started this task before the `reed-status-buttons` prototype.
This task builds the follow state, its rules above and the one follow button on the reed status bar.
The configurable button set stays with `reed-status-buttons`, which later takes the follow button into its set.
