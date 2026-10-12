# lyx-halt: ordered hub pause and resume

All six batches reported `done`, in plan order.

- **01-pause-record** (head `e3270c086`): the `internal/pauserecord` leaf with the pause record, coverage, intervals, sleep watch, `AwakeSince` and the network gate. The record path went into `hubgeom` and `standalonegeom`. Network git (`gitexec`) and GitHub calls (`githubclient`) wait at entry while a pause covers the process.
- **07-parentlock-covered** (head `b059edf46`): a covered run yields the parent lock queue. Shuttle starts, notices and the deadline honour the pause. An API-error stall across a detected sleep gets one automatic continue line. `run.json` names its step, and a stop verb ends one step's runs. Burler's live-fix resume delivers a pending task prompt. Runners are told their pause geometry.
- **13-shuttle-send-refusal** (head `ba9ad9d74`): `lyx shuttle send` refuses into a covered strand. Reed resume and batten's revive leave covered strands down. `lyx loom start`, `lyx loom resume` and `lyx batten run` refuse on a covered slug. Batten's idle clocks use the shared sleep watch, which fixes the eleven-hour notice. Loom keeps a covered holder's parent lock.
- **19-watchdog-awake** (head `a4ad225ab`): the step watchdog measures awake unpaused idle time, and a watchdog kill names itself and stops its step's shuttle runs. The orch watcher interrupts, holds and resumes under a hub pause. New `pauseengine` and `pausecli` packages carry `lyx pause`, `lyx resume` and the network gate install.
- **24-status-paused-since** (head `7aee839a9`): loom and batten status carry `paused_since`. A bare loom or shed pause of a loom run refuses with both ways forward. Busy and goto refusals name the recipe's pause, and the arbiter clause is gone. `pause --after` naming a waiting row ends its parent lock wait.
- **28-webster-pause-retire** (head `cbe02c166`): webster's pause verb, flag, `ErrPaused`, the `paused` outcome and the matching stencil and spec text are removed.

Reported deviations from the declared file-ops were additional files created or touched, such as `internal/shuttleengine/stepstop.go`, `internal/loomcli/pausehold.go`, `internal/battenshed/innerrun_pause.go`, the `internal/pauseengine` and `internal/pausecli` files, and the webster files that dropped pause wording. The test budgets for `battencli` and `loomcli` were raised, and four `PATTERN.md` entries were trimmed to fit the headroom lint after the orch-pane entry grew.

The batch 01 and batch 28 forks reported their card gates listing `./internal/gitexec_test` and `./internal/websterengine_test`, which are not directories. Those steps fail on setup, and the forks ran the real packages instead.
