# Add dobby, a per-beam daemon that drives runs; arbiter for all shed runs

Every beam, the hub's prime included, now gets `dobby`: a Go daemon with no LLM that starts the beam's ready tasks, drives their batten runs, lands them and cleans up.
It replaces the one waiting `lyx batten run` process per run.
It escalates only what needs judgement: first to the arbiter, then to the nearest live orch.
After a machine restart, one `lyx orch start` brings every run back.
The orch no longer starts, stops or cleans up runs.
It resumes a halted or paused run only by a hand pass, on the operator's word or after judging the stop.

## What changes

- **Dobby** lives in the beam module: `internal/dobbyengine` holds the daemon, `internal/beamcli` holds the verbs, and the hidden `lyx beam dobby` runs it.
  There is one per beam worktree on its owning machine, guarded by a lock and a state record under `.lyx/beam/`.
  Its creel sender is `DOBBY` (`ly:DOBBY` for the prime, `ly:<slug>:DOBBY` for another beam).
  It registers in `internal/runproc` as `RoleDobby`, so the pair teardown of a landed beam ends it, and `lyx reed down` of a beam's session spares it.
  The prime is the root beam on the same code path as every other beam; it differs only in having no parent.
- **`lyx beam start`** starts the dobby of every beam this machine owns whose dobby is not live, and is idempotent.
  `lyx orch start` calls it first (a failure is a warning, never a stop), and `lyx beam create`, `lyx beam promote` and `lyx adopt` end with it.
- **`lyx adopt` refusals**: adopting a task or beam whose parent another machine owns, or a beam with a not-done member another machine owns, is refused before any mutation.
  Each refusal names one call that satisfies both rules.
- **Batten runs in short passes.**
  Run-Shed's `InnerRun` makes one check of its child per call and returns a new shed outcome, `Yield`, which persists nothing and ends the step loop; the Go driver reads it as no stop.
  Dobby schedules the passes on child and batten status-file writes (`internal/fswatch`), on a probe tick, at its own start and at the lift of a tender pause.
  `InnerRun`'s in-memory latches (notice episodes, backoff, idle bounds, respawn counts) move to a per-run tolerant record, so once-per-episode rules still hold across processes.
- **Continue and full passes.**
  A pass dobby schedules into a run it already drives is a hidden `--continue-pass`, which skips the origin reconcile and the Open-Window re-run.
  The first pass after a dobby start or a machine wake, every pass while a full pass is owed, and every hand pass is a full pass.
  Dobby holds the sleep watch, an origin-owed marker keeps passes full while origin was unreachable, and a refusal naming another machine as owner has its own exit code that dobby handles as owned-elsewhere.
- **`draft` is a task status.**
  The draft rule lives in `Board.boardCriticalSection`: any write that leaves a task with an empty status gives it `draft`, except a release (`lyx board set-status '{"slug":"<slug>","status":null}'`) of an existing task.
  Cancel and `lyx fabric remove` leave a task `draft`.
  Batten's start guard refuses a draft, and the README marks drafts.
  A one-time migration marks open statusless tasks `draft` and records a `draft_status` key in `board.json`, without raising the on-disk version.
- **Dobby starts a task** when it is a task, its status is empty, its `depends_on` are done, its parent is the dobby's worktree, its owner is this machine or none, no tender pause covers it, and fewer than `max_runs` runs are in flight.
  New `beam.yaml` keys: `max_runs` (default 3), `board_tidy_every` (default 5, 0 off) and `board_tidy_timeout_min` (default 30).
- **Notices.**
  The landing notice no longer names newly ready tasks, the `landed` notice is gone, and the orch role stencil drops its start steps.
  A records stop after a successful landing still notices the orch.
  Land and Worktree-Teardown failure notices wait for the stop chain and post once the driver hands the stop back.
  Driver-dead and API-error notices follow the new respawn cap and the once-per-episode revive.
- **Arbiter for every shed run.**
  The spawn and prompt move from `internal/loomcli` into the new `internal/shedarbiter`, wired through `sheddriver`'s `Arbiter` seam for loom, darn and batten.
  The keys `arbiter`, `arbiter_timeout_min` and `arbiter_max_spawns` move to `shed.yaml`, with same-named overrides in `loom.yaml`, `darn.yaml` and `batten.yaml`.
  A config migration copies each key a hub layer sets in `loom.yaml` into `darn.yaml`.
  `internal/arbiterstop` gains a batten rule: an escalated tackler stop is eligible unless it is a hand-back (a landing halt, a spent budget, an unmet precondition or a pause-honouring Land stop).
- **Board tidy.**
  Every `board_tidy_every` counted landings, dobby spawns the board-tidy Sonnet agent and posts its report path to the nearest orch.
- **Finalize's per-worktree config notice** compares keys through `internal/configengine` and is silent when no key changes or the parent already holds the tip's value.
- `--window` and `--open-terminal` are retired; `open_window` alone decides the Open-Window row.
  `PATTERN.md` gains `PATTERN-dobby-starts-runs`, and `PATTERN-owner-machine` and `PATTERN-shed-stop-chain` change.

## Design decisions to review

- Tender stays the hub-wide brake, apart from dobby: dobby starts no task and runs no full pass under a covering pause, and a pause lift gets the covered runs a pass at once.
- Dobby finds its runs from local batten status files, never the board, so a run whose task was already pruned still reaches Worktree-Teardown.
- A pass refuses `--continue-pass` unless batten reads `running` at Run-Shed, the parent dobby's state names the run with no full pass owed, and no origin-owed marker stands.
- Dobby's own failures, a refused start and a pass that keeps failing, go to the nearest live orch with markers under `.lyx/beam/`; they are not shed stops.
- The one older-binary bound: a task an older binary creates on another machine after the migration has no status, so it starts without a release.

## To check by hand

- Not run here: the end-to-end behaviour of a dobby against real tmux and reed sessions, a machine restart followed by `lyx orch start`, and fswatch on WSL or through a junction.
- The hub's orch rule "Board tidy after a landing" on Bolt is left in place; remove it through `lyx orch rules` once this lands.
- Note `batten-window-default` is to be removed with `lyx board remove` (its idea is moot).
- Open finding, `forced-edits-outside-cards`: the edits some files needed outside the planned targets (listed under the batches' deviations in the run record) deserve a look for scope creep.
- Open finding, `draft-migration-after-mutation`: in `internal/boardengine`, check that the draft-status migration happens before the mutation runs, so a first-write release stays released.
- Open finding, `lift-full-pass-unreachable`: in `internal/dobbyengine`, check that the full pass at the lift of a covering pause is reachable and that a stale pending lift is dropped.
- The verify run found `TestCLIForm_EveryCommand` failing on `lyx orch start`'s `Long` help text over the size cap; the fix trimmed it in `internal/orchcli/start.go` (`28fe6e3f2`), so check the help still reads well.
