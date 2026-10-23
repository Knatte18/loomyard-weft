# One batten verb ends a run for good, live or stopped, before or after its pair exists

`batten-cancel-verb` · Task · bug, batten, reed, shed · running · Webster-Burler-Review

Imported from [issue #775](https://github.com/Knatte18/loomyard/issues/775).

Operator decision (2026-10-22): this task owns the one verb that ends a batten run for good, whatever its state.
It takes over the `batten-abandon` note from task `run-process-ownership`, which described the same verb, and folds in the note `batten-abort-live-run`, which extends it to a live run.
The verb ends a live run's processes through the owner record `run-process-ownership` builds.

## Note `batten-cancel-verb`

Seen on OSL-1046 (2026-10-20): two batten runs (watchdog-reap-test-xdr, lyxbin-timeout-test-xdr) were started from a prime with uncommitted changes. Worktree-Create went stuck on "source worktree has uncommitted changes", the driver spawned one tackler per run, and both runs ended blocked with no pair created. The operator then cancelled both runs.

There is no verb for that: lyx batten cleanup skips blocked runs, lyx batten pause only adds a pause, and lyx board set-status clears the board entry but leaves the batten records. So the reed overview keeps showing both as blocked, and the only way out is removing _lyx/shed/<slug> and .lyx/shed/<slug> by hand and committing the records side.

Two gaps:
- A cancel (abandon) verb for a batten run, at least one that has not created a pair, which removes its records and scratch and clears the board status.
- Worktree-Create's dirty-source stop is a precondition the operator or orch must fix, so it should halt with that way forward instead of spawning a tackler that cannot settle it.

### Operator decision (2026-10-20): batten refuses a dirty source up front

`lyx batten run` checks that the source worktree is clean before it does anything, and refuses with the dirty paths and the way forward when it is not: no run record, no pair and no tackler.
The orch should not have to run `git status` before every start; batten owns that precondition.
A dirty source never spawns a tackler, at start or at Worktree-Create.

Workaround: remove the two run directories by literal path and commit the records side with lyx fabric commit, done only while no other run is landing so the prime's records side is not dirty during a Land row.

## Note `batten-abandon` (moved from `run-process-ownership`, 2026-10-22)

### Seen

On 2026-10-18 the records held four batten run records with no pair and no board entry:

- `orch-rules-in-weft`, blocked at Worktree-Create because its slug ended in the reserved `-weft` suffix; the task reran as `orch-rules-files` and landed.
- `plan-gate-quality` and `fabric-net-and-config-warn`, paused at Run-Shed on 2026-10-08 a minute after their start, then re-scoped on the board into other entries.
- `webster-pack-end-of-prompt`, paused at Run-Shed on 2026-10-15, then replaced by `webster-pack-layout-switch`, which landed.

Their pairs were removed with `lyx fabric remove`, but no verb ends a paused or blocked run, and `lyx batten cleanup --apply` skips every paused, blocked, awaiting or failed run.
So the records stayed, showed as runs in the reed overview, and were removed by hand with `git rm` in the records repository.

### Idea

A batten verb, for example `lyx batten abandon <slug>`, ends a run that will not go on:
it refuses while the run's driver or run lock is live, tears down the pair if one is left, releases the run's board lock, removes the run's records directory and scratch, and commits and pushes the records.
It leaves the board entry itself alone, so a re-scope or rerun stays the caller's own step.

## Note `batten-abort-live-run`

### Seen (orch, 2026-10-21)

The operator wanted the live loom run `quarry-csharp`, at Discussion-Write, stopped for good and removed.
No verb does that, and the orch's manual stop took several rounds while the run kept coming back:

1. The orch killed the `lyx batten run` and `lyx batten step` processes and the Discussion-Write Claude session.
2. Reed relaunched the strand, since its tmux session still stood, so a new Claude session started.
3. Killing the tmux session on the reed server ended it, but a drive-report writer then started, and loom posted "Run quarry-csharp stopped".
4. `lyx batten status` still read `running` throughout, so the board claim stayed and the entry took no scope edits.
5. `lyx fabric remove` waits up to two minutes for the driver to go quiet, which looked to the operator like a hang.

`lyx batten pause` acts only at the next producer boundary, which a session waiting for its turn never reaches.

### Idea

One verb, such as `lyx batten abort <slug>`, that ends a live run of any state at once and for good:

- stops the batten and loom drivers, every strand of the run and its reed session, so reed relaunches nothing;
- writes no drive report and posts no stop notice beyond one abort notice;
- marks the batten record aborted and clears the board claim, so the entry takes edits again;
- optionally removes the pair, with `--remove`, without the two-minute wait since nothing is left to go quiet.

The `batten-abandon` idea refuses while the run is live and this one ends a live run, so the task settles one verb covering both, with the refusal or the abort chosen by a flag or by the run's state.
