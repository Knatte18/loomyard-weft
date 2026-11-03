# Drift runs in anchored hubs, can be skipped per run, and scopes smarter

Fixes three drift faults seen in a hub whose fabric is anchored at a subdirectory (issues #23, #24 and #25).

## The capture child starts in the anchored directory (#23)

The detached `lym drift capture-child` was started at the worktree root, and every lym verb refuses a cwd other than the anchored directory, so the child died before it recorded its start and drift could not run in any hub with a subdirectory anchor.
`SpawnRequest`, `CaptureRequest`, `CompareRequest` and the shared step runner now take an `AnchorDir` in place of `WorktreeRoot`.
Both callers, the Drift rows' wiring in `internal/loomcli/driftwiring.go` and the `lym drift` verbs in `internal/driftcli`, pass `location.AnchorPath()` from lymcwd, so the engine is told an absolute path and joins nothing by hand.
The capture child starts there, and a command step's relative `dir` is now joined onto the anchored directory too, so the capture and compare commands also start there.
In a hub anchored at `.` nothing changes.

## Drift can be switched off for one run, and a failed start is deferrable (#24)

The rows' `entry_gate: drift.enabled` is read only when the run first enters a row, so setting `enabled=false` in a task's local layer did nothing for a row already entered.
Drift-Baseline, Drift-Await and Drift-Verify now read `drift.enabled` at the start of every pass and return Done when it is off, without spawning, capturing or committing anything.
A resume after `lym config drift --layer local --set enabled=false` now passes all three rows.
A drift config that fails to load at that check is a Stuck naming `lym loom resume`.
Drift-Await needs the `Config` seam for this, and its registry entry now requires it.

A capture child that did not start used to come back from Drift-Baseline as an error, which left the run failed, and `lym drift defer` refused it.
It is now a Stuck that opens with `CaptureHaltPrefix` and names `lym loom resume` and `lym drift defer` followed by `lym loom resume`, so defer settles it.
Without `FallbackDetach`, `StartCapture`'s own error now names only the child's log, and the row adds the ways forward itself.

## The baseline agent checks the suite's reach, and the parent can fix the scope (#25)

The Drift-Baseline stencil gets a new `capture_command` marker, filled from the declared capture step.
The agent is told to follow the project references, imports or build dependencies of the projects that command runs, and to record scope `off` when none of them reaches a changed file, or when the task changes no code the suite runs.

`lym drift scope` now also accepts a call from the run's parent at a blocked Drift-Baseline or Drift-Await, and records or replaces the scope there.
It refuses while a capture holds the lock, and once `awaited_at` is set.
A replacement keeps the record's other fields.
When the new scope compares a test the old scope skipped, the replacement removes the capture record, since that capture has no profile of the test.
The resumed row then captures again, or halts with its usual ways forward.
The baseline agent's own second call at a running Drift-Baseline is still refused.

## Design decisions

- Command steps' relative `dir` now resolves against the anchored directory instead of the worktree root, as the parent's recorded decision requires.
  The only hub with a subdirectory anchor never got a capture to run, so no working drift config changes meaning.
- The switch-off check sits in each producer and not in the shed engine's entry evaluation, because re-evaluating every gate on resume would change every gated row in every recipe.
- Drift-Await's and Drift-Verify's passes with drift off commit nothing, as Drift-Baseline's does: no drift record changes on that pass.
- The gate-slot holder now names the anchored directory, since the step runner no longer gets the worktree root.
  The holder's worktree field is used only as a label in logs and wait records.

## Tests

- `TestStartCapture` checks that the child starts in `AnchorDir` and that the no-fallback failures name the child's log.
- `TestDriftBaseline`, `TestDriftAwait` and `TestDriftVerify` each gain a pass with drift switched off: Done, with no agent, capture, compare or commit.
- `TestDriftBaseline` also covers a failed config load, and checks that a start failure is a capture halt `Defer` admits.
- `TestRecordScope` covers the parent's record and replacement at both halts, the capture each replacement drops or keeps, and the refusals for a running capture, a settled wait and another row.
- `TestSeatTables` checks that the prompt shows the declared capture command, or says that none is declared.
- The registry entry test covers Drift-Await's new `Drift.Config` requirement.
