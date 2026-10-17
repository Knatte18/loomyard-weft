# Report timing: count a child's pause under its own paused phase

`lyx report timing` counted the time a run's child stood paused at a producer boundary as time of the phase of the row it paused at.
A run paused before Plan-Write on request reported hours of Plan time for sessions that ran minutes.
The report now counts that time under a new `paused` phase, apart from the phase it paused at, as it already does for machine sleep and waits.

Closes #695

## What changes

- `internal/reportengine` gains `PhasePaused` (`paused`) in the timing phase table, after `land` and before `other`.
  It flows through the run table, the bucket table and the medians like any phase, with no new figure or column type.
- A batten Run-Shed entry whose recorded change goes from `paused` counts its interval under `paused`.
  That interval runs from batten's change into `paused` to its change out of it, so it is the pause.
  Every other Run-Shed entry keeps its present phase.
- A run in flight whose last merged row is a Run-Shed change into `paused` counts its trailing interval to the snapshot under `paused`, not under the current producer's phase.
- A run's and a bucket's active seconds lose the paused phase's time as they lose waits.
  Net waits and paused intervals go through one union, so a wait overlapping a pause is taken out once.
  Phases plus sleep still sum to the span.
  Active, waits, paused and sleep sum to the span only when no wait overlaps a pause.
- A run with no change from `paused` holds `paused` as nil.
  The bucket table splits a pause across the buckets it overlaps.
- The `# Timing` section of `internal/reportengine/doc.go` and the report entry in `docs/overview.md` describe the new phase.

## Design decisions

- The change text is read only through `battenshed.ParseStateChange`, and the engine still reads no clock and derives no path; it reclassifies intervals it already computed.
- `paused` is the child shed's `paused` state, which batten's notice reports as `paused at Plan-Write`.
  A run that stands paused after a goto counts the time until its next step under `paused` too, as it is the same child state.
- Out of scope: the shed's own `activity.now`, so `lyx shed status` still shows `now: Plan-Write` during a pause, the convergence report, and hub-wide `lyx tender pause` entries, which reach no git record the report reads.

## Known limits

- A pause batten's poll never observed, such as one the child resumed from within one poll, leaves no change entry, and its time stays under the row's phase.
- A batten record written before change entries carried the `from` state reads no pause, and a run without the batten source is unchanged.

## Tests

`internal/reportengine/timing_test.go` covers the new behavior as rows and assertions in the existing `ComputeTiming` table test:
a landed run with a pause, a run in flight standing paused, a run with no pause, and a pause split across buckets.
The test tiers were not run by this description's author.
Check by hand: `lyx report timing --since 2026-10-14` against the twill-fork0 and kickstart-per-card runs from the issue, whose Plan time should drop and appear under `paused`.
