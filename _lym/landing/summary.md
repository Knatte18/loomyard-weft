# Drift: run in the anchored directory, skip when off, reset scope

Three drift faults surfaced in a hub whose fabric is anchored in a subdirectory by `.lym-anchor`.

Closes #23
Closes #24
Closes #25

## What changes

- **Capture and compare start in the anchored directory (#23).**
  The drift producer started the capture child at the worktree root, and the child refused to run there because its cwd did not equal the anchored directory.
  The capture child, the compare, the side capture and the detached start of the `lym drift` commands now run in the directory `location.AnchorPath()` resolves from `.lym-anchor`.
  The engine takes an absolute `AnchorDir` in place of `WorktreeRoot`, so no producer joins the anchor path by hand.
- **`drift.enabled = false` passes a row the run already entered (#24).**
  The entry gate reads `drift.enabled` only when a run first enters a row, so a local-layer switch-off did nothing after Drift-Baseline had failed.
  `DriftBaseline`, `DriftAwait` and `DriftVerify` now read the config on every pass and are Done at once, committing nothing, when it is off in any layer.
  A config that fails to load is a Stuck naming the error and `lym loom resume`.
  `DriftAwait` gains a `Config` seam for this.
- **Drift-Baseline halts name `lym drift defer` (#24).**
  A capture child that ended or stalled before recording its start is now a Stuck halt, with `lym loom resume` and `lym drift defer` followed by `lym loom resume` as the ways forward.
  Other start failures, such as an unresolvable binary, an unwritable child log or a held capture lock, stay errors.
- **The baseline stencil checks the suite's reach (#25).**
  `drift-template-baseline.md` now receives the declared capture command and its run directory.
  It tells the agent to follow the command's project references, imports or build dependencies and to record scope `off` when none reaches a changed file or the change touches no code the suite runs.
- **The parent can reset a wrongly recorded scope (#25).**
  `lym drift scope` is admitted for the run's parent at a blocked Drift-Baseline or Drift-Await, and replaces the record.
  The agent's own call stays once-only at a running Drift-Baseline.

## Design decisions

- Replacing a scope keeps the record's other fields.
  It is refused while a capture holds the lock, and once the baseline wait is settled (`awaited_at` set).
- A replacement that compares a test the old capture skipped drops the capture record, since that capture holds no profile of the test.
  The drop runs before the scope is written, so a failed drop leaves the old scope and the call can be repeated.
- A capture child's start failure is a typed `childNoRecordError`, so the row tells a child that ran and failed, which is a capture halt, from an environment fault, which is an error.
- `gateslot.Holder.Worktree` may now be the anchored directory, and its comment says so.

## To check by hand

Not run against a real subdirectory-anchored hub such as NORCE-DrillingAndWells/Models; the new tests cover the anchor in `driftcli` and `loomcli` integration tests and the engine rows with fakes.

- Run a darn task in a hub with a subdirectory anchor and confirm that Drift-Baseline starts the capture child and records its start.
- Set `lym config drift --layer local --set enabled=false` at a failed Drift-Baseline, run `lym loom resume`, and confirm Drift-Baseline and Drift-Verify pass.
- At a blocked Drift-Baseline or Drift-Await, run `lym drift scope` a second time and confirm the replacement, and that it drops a capture that skipped a test the new scope compares.
- Read the new `drift-template-baseline.md` wording as the baseline agent would, and confirm the capture command renders where the stencil places it, including the not-declared case from `captureReading`.
