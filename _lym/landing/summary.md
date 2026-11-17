# The dobby tears down or reports leftover pairs

A landed pair, a bench pair, a kept pair and a trial pair stood for days in the operator's VS Code with nothing reporting them.
The dobby now tears down the pairs no run's own teardown takes, and reports the ones it keeps.

## What changes

- **Landed and cancelled runs.**
  The dobby tears down, through `internal/pairteardown`, the pair of each run in FSM state `landed`, or `cancelled` without keep.
  A pause, a hold or another owning machine stops it.
  A cancelled run's batten record goes in the same step.
  This replaces the board-entry-based stranded-run teardown in `internal/dobbyengine/ended.go`, now `leftover.go`, whose condition reads the FSM state.
- **Handed-over pairs.**
  The dobby removes, in `pairteardown`'s local mode, the local pair of a run this machine handed over while another machine owns it.
  A hold does not stop this, since the local mode takes no origin step.
  A machine that re-adopted the run keeps its pair.
- **Kept pairs.**
  A cancelled run whose pair was kept stays.
  `lym beam status` lists it with its age, and each dobby start posts one notice naming `lym fabric remove <slug>`.
- **Pre-upgrade kept cancels.**
  A run cancelled with keep before cancel intents existed has a batten record in shed state `cancelled` and no cancel intent, so `runfsm.Derive` reads it as neither cancelled nor kept.
  The first dobby start writes it a finished keep cancel intent naming this machine (`battencli.MarkKeptCancels`, `shedrun.WriteKeptCancel`) and pushes it through the synchronous records push.
- **Orphans.**
  A pair that neither a board entry nor a batten record names is an orphan.
  The dobby posts a notice at first sight saying when it goes.
  It tears the pair down once `dobbyengine.OrphanGrace` (24 hours) has passed since the notice and since the last commit on either branch, and no process of the slug lives.
  Uncommitted files are committed onto the archived tip first, and the branches are kept under archive tags (`pairteardown.RemoveOrphanPair`).
  The first-sight time lives in a machine-local file, `fabricengine.OrphanFirstSightPath` under the new `lymdirs.OrphanDirName`, so a dobby restart neither resets nor skips the grace.
  The file is dropped with the teardown or once the pair stops being an orphan.
- **`lym tender resume`.**
  It spawns no detached pair push for a transition, since every transition pushes synchronously.
  It still pays a push a pause deferred, such as an anchor change's.
- **Hub workspace file.**
  `fabricengine.RegenerateHubWorkspace` runs after each teardown.

## Design decisions

- A failed teardown retries with a wait that starts at one probe interval and doubles to a ceiling, and logs once per failure episode.
- An orphan is re-checked just before its teardown, which refuses when a board entry, a batten run directory or a live process now names the slug.
- A kept pair's batten record keeps it out of the orphans, also when its board entry is removed.
- `pairteardown` gains the local-mode and orphan removals; `PATTERN-pair-remote` records that D7's teardowns go through it.
- The reed engine's binding tests wait up to 30 seconds for the recorder script, since the tmux spawn timed out at 5 seconds under the package's parallel load.

## To check by hand

- Not run: a live dobby over a real hub, observing a landed pair, a cancelled pair, a kept pair and an orphan across a restart.
- The 24-hour orphan grace and the notice wording are covered by tests with a fake clock only.
