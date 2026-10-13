# Logging cleanup: level policy applied, reed resize fixed, logs archived

## What changes

lyx's logs now follow the level policy in `internal/logger/logger.go`: an ordinary state logs once at Info or Debug, and nothing repeats per tick or per poll.
In the prime's traces about 80 % of WARN lines recorded ordinary waits, and one reed line made up most of the volume.

- **Reed resize failure.** The hub watchdog kept running the `lyx` binary it started from after a deploy replaced it.
  That stale render left the top-anchored `loom-status` strand out of the layout, so tmux refused every re-apply.
  The watchdog now re-stats its executable on its existing discovery tick; when the file changed it releases the lock, spawns one successor through `SpawnWatchdog` and exits 0.
  A failed stat counts as unchanged, and nothing is spawned under `go test`.
- **Once per episode.** `logger.Episode` and `EpisodeSet` log a repeating failure as one Warn, then Debug while it persists, then one Info on recovery.
  Reed's resize re-apply, watch-tick and name-repair failures, battenshed's polled seam failures, shuttle's polled failures, orch tick failures, the loop watchdog and waiter reads, the idle-release hook and the parent-lock pause-flag read use it.
- **Batten waits.** `reportStuck` takes its level from the cause: Warn for a failed Worktree-Create, Seed-Child, Land or Worktree-Teardown step, Info for waits, state changes, pauses, blocks and halts.
  The reason file is still written on every change; the log dedupes on a key with counters and error tails left out.
- **Orch.** Benign idle-probe disagreements (`busy`/`background` beside an idle pane, `asking` beside a non-idle pane) now count as agreement; others log once per episode.
  Hold reasons carry a constant "the input box holds a draft" instead of the operator's draft text, so the dedupe holds and the draft stays out of the trace.
  Shuttle exposes a draft kind through its provider-neutral types for this.
- **Smaller lines.** A lost watchdog lock race logs at Debug, so it writes no trace file at the default level.
  A missing transcript for the session name logs at Debug, and other read failures Warn once per session.
  `battenshed: session state at notice` is Debug, and `battencli: create worktree` no longer repeats the mutations.
- **Test noise.** The logger's stderr half is discarded under `go test`; a test that asserts on log output calls `logger.SetOutput` itself.
  The gate helper in `internal/gatecli` does so to stand in for `lyx`'s main.
- **Log archive.** Pair teardown copies the pair's trace logs and `verify-*.log` files to `<hub>/_board/.lyx/log-archive/<slug>/<UTC timestamp>/` before removal, and prunes entries older than 30 days.
  `fabricengine.LogArchiveDir` names the root.
  A copy failure logs Warn and never fails the teardown.
- **Landing notice.** Every landing sends `<slug>: landed`, with `; newly ready: …` appended when it applies.
  The orch stencil routes every landed notice to After landing and catches a dropped one at its next wake from the board and the notice log.
- **Docs.** The Level policy gains three rules, with a new `PATTERN-log-levels` entry; `doc.go` files, `PATTERN-pair-teardown` and `docs/overview.md` follow the changes.

## Design decisions

- The retention is a Go constant, not a config key; a hub-wide key would need a new module with no one asking to tune it.
- The log audit covers the sites the plan and its reviews listed.
  Per-tick lines found later, below further callees or behind callbacks, are not defects of this change.
- `gitrepo: push refused, retrying` stays as it is: a recoverable retry fits the policy.

## To do or check by hand

- **Operator step after deploy.** A watchdog started before this lands has no binary check and holds the hub lock, so the resize failure goes on.
  End each running pre-change `lyx reed watchdog` once; the next `reed up`, `attach` or `resume` spawns one that retires itself on later deploys.
  Afterwards the hub watchdog's trace should show no `resize re-apply failed` line for a session holding a `loom-status` strand.
- **Windows is unverified.** A running binary cannot be overwritten in place there, so the retirement check fires on whatever the deploy leaves at the path, or not at all.
- **Decision-read guard rows.** In `internal/battenshed/innerrun.go`, the decision read in `handedLockStep` is deliberately not guarded, since `awaitingStep` ends the Call on the same check.
  The review found the stale-base and notice-step decision-read rows of `TestInnerRun_ReadFailuresLogOncePerEpisode` unreachable; confirm they exercise a guarded path.
- **Recovery branches.** Several recovery branches of the once-per-episode guards are not reached by any test; check them by reading.
- **Dropped landing notice.** The orch's recovery from a dropped landing notice rests on the stencil's wake-time board checks, which no test runs.
- **Interplay with in-flight tasks.** `go-driver-assistant` may touch `internal/battenshed/innerrun.go`, and `run-window-bugs` changes shuttle's readiness and send paths; whichever lands second merges against this.
  If the assistant keys on WARN lines to find a stop's cause, the batten waits that moved to Info fall outside it.
- **Archive size.** Verify logs were never measured, so the archive's size rests on the 30-day retention alone.
- **Gate fix.** Two `internal/gatecli` integration scenarios failed because the logger now discards stderr under `go test`; `runGateHelper` calls `logger.SetOutput(os.Stderr)` and only `./internal/gatecli` was re-run.
  Other packages with helper binaries that read logger output from stderr were not checked.
