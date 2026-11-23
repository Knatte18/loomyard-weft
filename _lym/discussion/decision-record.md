## Goal

````markdown
Imported from [issue #198](https://github.com/Knatte18/loomyard/issues/198).

The `Code-Burler-Review` seat halted the run as `producer-hard-failure` at round 1 attempt 1b, before any review ran:

```
review: stencilstore: stencil "burler-assist-consolidation" was written by lym build 8cf7f477563f044e178a070cfd7dd27cb9f03c78, which is newer than the running build 52d132a2c8e4e3b68e3e797e30d3e9ceb200ec50, and this process may not render it; way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board
```

The run's driver process had been started on an older lym build than the one that wrote the stencil on the board.
The run had been going for over thirty minutes (Preflight at 08:25Z, halt after Code-Bouncer at 08:59Z), so the board was upgraded mid-run while the driver kept the old binary.

This is abnormal because the guard is correct but its result is a hard `failed` halt of a long-running run, with the recovery left to the operator.
Nothing warns earlier that the driver binary has fallen behind the board.

What to do: consider detecting the build skew when the driver steps between segments and halting as a resumable state, not `failed`.
Alternatively, have the driver re-exec itself on the current binary, or surface the skew in `lym loom` status before a seat needs the stencil.

Provenance: task `godocreflow-worsens-findings`; notes `loom-halt.md`.
````

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Narrow to the false hard-failure report

Decision: Option (b): a stop the driver hands over to a fresh build is not filed as producer-hard-failure; it gets no halt note, or a distinct anomaly kind the selfreport path does not turn into an issue. No pre-step hand-over (option c).

Rationale: The existing hand-over already recovers the run; the remaining harm is the false halt note, which became issue 198.

### Added after Discussion (arbiter, 2026-11-23): Every failed halt on a newer board stencil is build-skew

Decision: Keep failedHalt in internal/loomcli/halt.go as written: a failed halt whose error chain holds a stencilstore.NewerBoardCopyError is classified build-skew whether or not the driver later hands the run over to a fresh build. Do not narrow the kind to the hand-over case.

Rationale: The board entry calls the stencil guard correct; the harm in issue 198 was reporting the skew halt as producer-hard-failure. A skew halt the driver does not hand over (no newer build on disk, or its one hand-over spent) has the same cause, a lym binary to update, not a lym defect, and its tackler settles the stop. The halt note is written before the driver decides, so narrowing would need knowledge the note writer lacks. This applies the Narrow-to-the-false-hard-failure-report entry to the same error without widening the task's scope.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
