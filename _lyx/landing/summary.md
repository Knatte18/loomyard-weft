# Report timing: count a child's pause under its own paused phase

`lyx report timing` counted the time a run's child stood paused at a producer boundary under the phase of the row it paused at.
A run paused before Plan-Write for an hour and a half reported that time as Plan, even though the Plan-Write session ran for minutes.

## What changes

- A new phase, `paused`, joins the timing phase table, after `land` and before `other`.
  It flows through the run table, the phase medians and the bucket table like every other phase.
- Batten records every child state change as a Run-Shed history entry.
  The interval that a change out of `paused` closes is the pause itself, and it now counts under `paused`.
  Every other Run-Shed entry keeps the phase it had.
- A run in flight whose last row is a change into `paused` counts its trailing interval to the snapshot under `paused`, not under the current producer's phase.
- A run's and a bucket's active seconds now leave out the paused phase's time along with the waits.
  They are taken out through one union, so a wait that overlaps a pause is subtracted once.
- The package doc's `# Timing` section and the report entry in `docs/overview.md` describe the new phase.

## Design decisions

- Pause is a phase rather than a wait kind: waits stay inside their phase's time, so a `pause` wait would have left the time under `plan`.
  A phase also needs no new per-run or per-bucket field.
- Only an entry that `battenshed.ParseStateChange` reads as a change from `paused` moves to the new phase.
  A run without the batten source records no pause and times exactly as before.
- A hub-wide `lyx tender pause` is out of scope: its record is ephemeral and reaches no git record that the report reads.
  The shed status's `now: Plan-Write` during a pause is also unchanged.

## Tests

`TestComputeTiming` gains two rows.
The first is a landed run paused before Plan-Write across UTC midnight, with a parent-review round that overlaps the pause.
It pins the phases, the active figure with the overlap taken out once, the day-bucket split and the `paused` median.
The second is a run in flight that stands paused and trails under `paused` to the snapshot.
