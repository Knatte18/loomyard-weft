# Every run process has one owner record that the end verbs use

Processes lyx starts for a run now belong to an owner record, so `lyx tender status` lists them and the verbs that end or tear down a run end all of them.
On 2026-10-20 a paused run's driver chain and parked pushes outlived their pairs by 19 hours, and `lyx fabric remove` hung on them until the operator killed them by hand.

Closes #760
Closes #796

## What changes

- **Owner record.** The new leaf package `internal/runproc` keeps one JSON file per process under `<HubScratchDir>/runproc/<owner>/`, where the owner is a run's slug or one reserved hub-scope name.
  Detached pair pushes register under the pair's slug.
  Prime records pushes and board syncs register under the hub scope.
  `lyx loom drive`, the detached step loop and a seeded `lyx batten run` register themselves under their slug.
  Reads prune records whose pid is dead or whose start time differs.
- **`runproc.End`.** It is the only way to end a run's processes.
  It signals recorded groups, non-leaders and the descendants read before the first signal, escalates to SIGKILL after five seconds, and returns an error naming the pid and role of any survivor.
  It spares the caller and its ancestors.
  On Windows it ends no recorded pid, because process start times are not readable there.
- **Pair teardown and `lyx reed down`.** `internal/pairteardown` ends the pair's owned processes before the loop end and the session end.
  `lyx reed down` in a pair does the same on local readings only and makes no network call.
  It now refuses a run that reads running and unpaused, with the way forward `lyx tender pause <slug>`.
  It used to kill the session and let batten revive the strands.
- **`lyx fabric remove`.**
  It runs the run-scope origin reconcile first.
  A `stopped` verdict refuses.
  An `ended` or `owned_elsewhere` verdict removes the pair in local mode: no archive tag, no remote branch step, no board write.
  The quiet wait's driver checks are skipped for a paused, ended, foreign or locally done run with batten no longer running.
  The run-lock wait still runs first, so a step doing local work finishes before anything is ended.
  A holder still there at the bound is ended only if it waits at the network gate (a new waiting marker written by `pauserecord.Network`) or waits on its interrupted agent under a covering pause; any other holder refuses with `ErrDriverBusy` naming its pid.
- **Pause and lift.** Detached pushes and board-sync children exit at once under a covering pause, keeping their commits and pending flags.
  A push a Finalize spawns past its parent-side merge is not deferred by an owner pause.
  Every lift path in `tenderengine` (`resume`, hibernate wake and cancel, `lyx adopt`, cleanup reconcile) spawns the pushes and board sync the pause deferred, except for removed pairs and pairs another machine owns.
- **Owned-elsewhere pauses.** The hibernate wake, `lyx orch start` and batten's owner probe end the run's processes on the machine the run left.
  A run whose Finalize is past its parent-side merge is not ended and finishes its landing.
- **`lyx tender status` and `lyx tender reap`.** Status lists owned processes per owner and orphans.
  An orphan is a registered process of a slug with no pair and no live registered batten, or an unregistered lyx process whose working directory is a deleted directory inside the hub and outside every registered worktree.
  `lyx tender reap` ends exactly the listed orphans, exits non-zero if one survives, and refuses an agent caller.
- **Hibernate confirmation.** A session busy only on shells counts as settled.
  A fork still withholds, for up to sixty minutes in all; other reasons fail at ten.
  The requesting orch's own session is left out of the reading.
- **Shuttle (#796).** A gated run's waiting turn end that holds no fork and no awaited shell is an arrival, so a passing gate finishes the run Done.
  An ungated run with missing outputs and only such shells stalls after `Config.BackgroundShellWaitMin`, so the row's stall handling surfaces it to the parent.

`PATTERN-run-process-owner` is new, `PATTERN-pair-teardown` and `PATTERN-leaf-packages` are updated, and the package docs and `docs/overview.md` follow.

## Design decisions to know

- A pause still parks the Go driver, the step loop, batten and a session's shells; only detached pushes and board syncs stop.
  The parked chain carries a resumed agent's completion, and every end verb now reaches it.
  This departs from the board entry's "a pause stops work instead of parking processes".
- Detached children of an interrupted CLI call are not tied to that call; the owner record reaches them instead.
- A killed batten leaves its run record reading `running`.
  Until `batten-cancel-verb` ends that record, readers count batten as running only while a process registered under the slug with the batten role is alive.
  The board claim settle for a `running` record with no live batten keeps the claim and names `lyx board set-status`.
- `lyx reed down` in a pair no longer ends the session of a running unpaused run.
- Reed's own session end is unchanged.
- Processes an older binary started are never registered.
  The driver-lock re-check refuses a `RefuseWhenBusy` teardown that would outrun them, and the deleted-directory rule in status finds the rest.

## Check by hand

- `runproc.End` in `internal/runproc/end.go`: whether it watches every live member of a signalled group, and reports a failed signal's members, after the fix.
- `PATTERN.md` and its overview lines: whether the entries unrelated to this change are still as on `main`, after the headroom shortening.
- The run-lock wait in `internal/pairteardown`: whether the refusals that should pass before the wait are reached.
- The `lyx reed down` refusal text in `internal/reedcli/up.go`: whether its retry step matches the order the teardown actually runs.
- Not run: the end-to-end incident (paused run with parked driver, loop, batten and pushes, then `lyx fabric remove`) on a real hub, and the Windows paths, where `End` ends no recorded pid.
