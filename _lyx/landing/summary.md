# Loop shed steps in Go; wake the loop driver only at a stop

The loom driver now steps in only when something goes wrong.
`lyx shed step <run-id> --until-stop` (and so `lyx loom step --until-stop`) runs one step after another and prints one envelope at the stop, so a driver's context stays small across a whole run.
A failed step no longer parks silently, and a driver woken at an awaiting PR-Gate steps the run instead of judging the pull request on GitHub.

Closes #488
Closes #491

## What changes

- **Loop mode of `step`.**
  Each iteration runs the single-step verb as a child process with its own trace id, so per-step hooks, records and traces stay what a single `step` does today.
  The loop stops at a halt (`blocked`, `awaiting`, `paused`, `done`), a step error, a child that ended without an envelope, or a fired stop condition.
  The final envelope is the stop step's short envelope plus a closed-key `loop` object (stop cause, detail, trace copy, stderr path, status movement, interrupt policy, re-step).
- **Detached loop, waiting driver command.**
  `--until-stop` spawns the loop detached (`proc.DetachBreakaway`) and waits for its envelope file, so a killed driver shell does not take the loop.
  A loop lock in the run's steps directory is the liveness record; a second invocation waits on the live loop, and a dead loop is reported as an `interrupted` stop with cause `loop-exited`.
- **Stop conditions.**
  `lyx shed pause <run-id> --before <producer>`, `--after <producer>` and `--clear` record a condition in the status file, and `internal/shedengine` honours it at the boundary check.
  Bare `pause` keeps its meaning; `--clear` never removes a bare pause request.
- **Step watchdog.**
  A child with no activity for `step_idle_timeout_min` (new `loom.yaml` key, default 60, floor 1) is killed with its process tree and reported as `interrupted`.
  Activity is the newest of the step's trace group and the run's agent activity, counting every seat strand of a multi-seat row.
- **Rich stops, short successes.**
  A stop carries the step's error, transient class, WARN/ERROR trace lines and the paths to read further; the stop step's trace is copied into the steps directory so retention cannot sweep it.
  The step envelope's `progress` drops `remaining` and empty keys.
- **Failures are visible.**
  The loop writes `failed` through one `internal/shedengine` function when a stop leaves the status file `running`, so `lyx loom resume` never answers "already running" for a stopped run (new `PATTERN-failure-stop-visible`).
  A pre-step arming refusal now carries `kind: bootstrap` and a trace, and the missing-hub-config case has a refusal row.
- **One transient re-step in Go.**
  The loop re-steps a transient failure once, moved from the stencil.
- **Driver stencil.**
  Rewritten around the loop: no per-step trace reads, one fork at an error stop, the repair catalogue moved to the new `shed-template-driver-guide` stencil, and an awaiting PR-Gate steps the run without inspecting the pull request.
  `scribe:prose` is dropped from the driver's skills, and the orch stencil gains one line on asking a driver to stop at a point.
- **Teardown.**
  `internal/pairteardown` counts a live loop as busy, and session end kills the loop and its step tree first.
  `lyx loom start` keeps the status strand on llm-driven runs.
- **Trace noise.**
  The plan gate logs informational findings as a count plus a file, and the stencil drift notice is one warning per process.

## Design decisions

- The loop is a mode of the generic verb in `internal/shedverbs`, so every subtree that arms the verb gets it.
  `PATTERN-shed-verb-set` changes: the step kind vocabulary gains `interrupted`, and the package's import allowlist gains `internal/proc` and `internal/fswatch`.
- The child tie (parent-death signal on Linux, kill-on-close Job Object on Windows) moved from `internal/gatecli` into `internal/proc`, and `lyx gate test` uses it from there.
- On Windows the loop's identity is a named Job Object, since `proc.StartTime` reads nothing there; on Linux a kill acts only while the loop lock is held and the recorded start time matches.
- The loop never re-invokes an interrupted step; the driver's interrupted rule decides from `loop.interrupt_policy` and `loop.status_moved`, under its repair cap.
- Out of scope: the hub-wide interrupting pause, message routing, changes to the Go driver `lyx loom run` beyond honouring stop conditions, and lower effort for Sonnet drivers.

## To check by hand

- Session end can respawn a loop in the window between its kill of the loop and the run lock's release (`session-end-respawn-window`, left open by review): check the teardown order in `internal/pairteardown` and the session-end step.
- Nothing here was run on Windows: check `proc.DetachBreakaway`, the named Job Object kill and the kill-on-close child tie there.
- Run a real loom run with `--until-stop`, kill the driver shell mid-run, and confirm the loop survives and the next invocation prints its envelope.
- The 60-minute idle default is a guess; a legitimately silent step (a long git transfer, a stale verify wait marker) may be killed, so watch the first runs.
- `history_length` is not a strict move counter, so an unwatched envelope whose producer, history length and state all recur after a resume can be delivered although stale.
- `internal/proc/tied.go` cannot import `internal/logger` without an import cycle, so it is listed in `spawnObservabilityAllowedSpawners` in `cmd/lyx/spawnobservability_test.go` with a reason: check that exemption is acceptable.
- Two tests were adjusted to fit the change and deserve a look: `stepDriftWarningNeverBlocksExitCode` in `internal/stencilcli` now asserts the shared "drifted from worktree source" fragment of the one-line warning, and `envkit.FullEnv` in `internal/testkit/envkit` now sets `RunScratchDir`.
- The fork's context cost depends on the provider's cache behaviour; if a fork costs more than reading the trace in place, the stencil should read small traces in place.
