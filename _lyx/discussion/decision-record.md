# Decision record — timing-pause-as-phase

## Goal

`lyx report timing` counts the time a run's child stood paused at a producer boundary under its own phase, `paused`, and never under the phase of the row it paused at.

## Scope

- `internal/reportengine`: the timing phase table, the row-interval phase assignment, the trailing interval of a run in flight and the active figure, per run and per bucket.
- The `# Timing` section of `internal/reportengine/doc.go`.
- `internal/reportengine/timing_test.go`.
- Out of scope: the shed's own `activity.now` during a pause (`lyx shed status` showing `now: Plan-Write`), the convergence report, and hub-wide `lyx tender pause` entries, which live in the ephemeral pause record and reach no git record the report reads.

## Decisions

### Paused is a phase of the closed table

Decision: `PhasePaused` (`paused`) joins the timing phase table, after `land` and before `other`.
Rationale: a phase flows through the run table, the bucket table and the medians with no new figure or column type, and phases plus sleep keep summing to the span.

### The paused interval is the one batten's change out of paused closes

Decision: a batten Run-Shed entry whose recorded change goes from `paused` counts its interval under `paused`.
Every other Run-Shed entry keeps its present phase.
Rationale: batten records every child state change, so the child's pause runs from the change into `paused` to the change out of it, and the interval ending at the change out of it is exactly that pause.
Bound: only an entry `battenshed.ParseStateChange` reads as a change from `paused` moves; a run without the batten source has no such entry and its time is unchanged.

### A run in flight that stands paused trails under paused

Decision: in a run in flight whose last merged row is a Run-Shed change into `paused`, the trailing interval to the snapshot counts under `paused`, not under the current producer's phase.
Rationale: such a run has not resumed, so the time since the change is pause time.

### Paused time is not active time

Decision: a run's and a bucket's active seconds lose the paused phase's time as they lose the waits, through one union of the net waits and the paused intervals, so a wait overlapping a pause is taken out once.
Rationale: a paused run is doing no work, and the issue asks for pause time reported apart like sleep and waits.

## Constraints

- `PATTERN-told-geometry`: the engine reads no clock and derives no path; the change only reclassifies intervals it already computes.
- `PATTERN-sole-parsers`: the change text is read through `battenshed.ParseStateChange` only.
- `PATTERN-test-economy`: the new behavior is covered as rows or assertions in the existing `ComputeTiming` table test.
- The package doc's `# Timing` section is updated in the same commit as the code.

## Auto-mode assumptions

- The pause in the issue is the child shed's `paused` state, which batten's notice reports as `paused at Plan-Write`, not a hub-wide tender pause.
- A run that stands paused after a goto counts the time until its next step under `paused` too, since that is the same child state.

## Open risks

- A pause batten's poll never observed, as when the child resumed within one poll, leaves no change entry, and its time stays under the row's phase.
- A batten record written before change entries carried the `from` state reads no pause.

## Acceptance criteria

- A landed run whose batten record holds `child running → paused at Plan-Write` and later `child paused → running at Plan-Write` counts the gap between those two entries under `paused`, and its `plan` phase holds only the time outside that gap.
- That run's active seconds exclude the paused time, and its phases plus sleep still sum to its span.
- A run in flight whose last row is a change into `paused` counts its trailing interval to the snapshot under `paused`.
- A run with no change from `paused` holds `paused` as nil.
- The bucket table splits a pause across the buckets it overlaps and leaves it out of the bucket's active seconds.
