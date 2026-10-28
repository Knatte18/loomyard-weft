# Decision record — spawn-text-file-busy

## Goal

A loom run no longer ends `failed` because the `lym` binary was being replaced at the moment a loop child step was started, and when a child start does fail the halt note explains why on its own.

## Scope

- The loop's child start in `internal/shedverbs` retries a start refused with ETXTBSY a bounded number of times with a short, doubling backoff before reporting `start-failed`.
- `internal/proc` gains the ETXTBSY predicate beside its EAGAIN one.
- `tools/deploy`, which `update-plugins.sh`, `deploy-dev` and the hub's Land-row `deploy` command all run, builds `lym` into a temporary file in the install directory and renames it over the target.
- A `start-failed` interrupted stop names the attempts and what ETXTBSY means in its cause, and its trace file falls back to the loop process's own trace file, which holds the start failure, when the child left no trace and no stderr.

Out of scope: every other spawn site of `lym`, and a new anomaly kind or transient class for a failed start.

## Decisions

### D1 — Retry the loop child's start on ETXTBSY

Decision: `runChild` starts the child again when the runner's error is ETXTBSY, five tries in all, waiting 100 ms before the first retry and doubling after, through the loop's existing `LoopSpec.Sleep` seam so a test records the waits.
Each retry is logged at Warn; any other start error fails at once as before.

Rationale: ETXTBSY is refused before the child runs, so a retry never runs a step twice, and the bound of about 1.5 s covers a binary copy while still failing a binary held open for writing.

### D2 — ETXTBSY predicate in proc

Decision: `proc.IsSpawnBusy` reports whether an error is ETXTBSY, the one predicate for it.

Rationale: `proc` already holds the EAGAIN start predicate, and the loop imports it.

### D3 — Atomic binary install

Decision: `tools/deploy` builds into a temporary file beside the target in the install directory and renames it over the target, removing the temporary file when the build or rename fails.

Rationale: `go build -o` copies into an existing target in place when its work directory is on another filesystem, which is the window a spawning process sees as ETXTBSY; a rename on one filesystem replaces the directory entry atomically, and a running process keeps its old inode.

### D4 — Self-explanatory start-failed stop

Decision: a start that still fails ends the cause as `start-failed after <n> tries: <error>` and, for ETXTBSY, adds that the executable was open for writing, as while a deploy replaces it in place.
`interruptedTraceFile` falls back to the loop process's own trace file when the child left neither a trace nor a non-empty stderr.

Rationale: the loop's own trace holds the start warnings, so the halt note's `trace_file` is never empty when the loop has a trace.

## Constraints

- `PATTERN-spawn-observability`: every retry logs through `internal/logger`.
- `PATTERN-test-speed`: retry waits are injected through `LoopSpec.Sleep`, never sat out.
- `PATTERN-lym-temp-root` covers `lym`'s own temp files; the deploy tool's temporary binary sits in the install directory because a rename across filesystems is not atomic.

## Auto-mode assumptions

- The Land row's `deploy` command runs `update-plugins.sh` or `tools/deploy`, so fixing `tools/deploy` covers it.
- On Windows a rename over a running `lym.exe` fails as `go build -o` over it did before; the change keeps that behaviour.

## Open risks

- A deploy that holds the binary open for writing longer than the retry bound still ends the run `failed`, now with a cause that names it.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - start-failed-cause-not-decided-form (class design, severity BLOCKING, open in rounds 1)
  - trace-fallback-requires-start-failure (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1, D2: a child start refused with ETXTBSY and then accepted runs the step, and the recorded waits double from the first delay.
- D1, D4: a child start refused with ETXTBSY on every try stops interrupted after the bounded tries, with a cause naming the tries and the ETXTBSY meaning.
- D1, D2: a child start refused with another error is not retried.
- D4: an interrupted stop whose child left no trace and no stderr records the loop's own trace file.
- D3: `tools/deploy` leaves the binary at the target and no temporary file in the install directory.
