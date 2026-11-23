```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced the diff's changes to the board entry's goal points and the record's added entries.

**Decisions**
- **G1** (board entry `## Goal`): in a hub whose scoper cannot run tests on the merge base, the gate skips red-first, or runs it through the hub's own test command, instead of halting every run for a manual waiver.
  - Built by C1, so it is built: the diff takes the "skips" branch of the goal's either/or.
- **Added after Discussion entries:** none. The record's sections after `## Goal` are empty.

**Changes**
- **C1** traces to G1. `internal/loomshed/redfirstgate.go:216-219` makes `verify` return a passing verdict for `gateslot.RedFirstUnjudgedExit` when the run did not time out and the runner did not crash.
- **C2** is forced by C1. Once exit 5 passes in `verify`, these paths never see it, so the exit-5 handling there is dead:
  - `holdsForCommitOrRecord` drops exit 5 (`redfirstgate.go:226-231`);
  - `redFirstHoldLine` drops its exit-5 branch (`redfirstgate.go:234-238`);
  - `redFirstFailure` drops its exit-5 case (diff piece 004, the removed hunk near old line 268).
- **C3** is forced by C1. The doc comments of `NewRedFirstGate` and `verify` now describe the new verdict (`redfirstgate.go:41-43`, `:201`).
- **C4** is forced by C1. `internal/gatecli/redfirst.go:129` changes the verb's exit-5 message. The old way forward asked for a `No test seam` record, which the gate no longer needs, so that wording would have been wrong. The exit code itself does not change.
- **C5** is forced by C1. Four doc comments described exit 5 as a hold or waiver and now say it passes:
  - `internal/gatecli/doc.go:472-474`
  - `internal/gateslot/redfirst.go:16`
  - `internal/loomshed/doc.go:69-70`
  - `internal/shedrecipe/doc.go:63-66`
- **C6** is forced by C1. The darn writer stencil told the writer that a non-go hub holds the handoff and to ask for a waiver. It now says the gate passes there (`contracts/stencils/darn/darn-template-write.md:111-113`).
- **C7** is forced by C1. The tests follow the changed behaviour:
  - the two exit-5 verdict rows now expect a pass (`internal/loomshed/redfirstgate_test.go:118-124`, `:142-149`);
  - the hold test drives its hold with exit 4, the no-test exit, in place of exit 5 (diff piece 004, test hunks at old lines 345 and 375);
  - the integration test asserts the new message (`internal/gatecli/redfirst_integration_test.go:153-154`).

**Findings:** none. Every change traces to G1 or is an edit C1 forces, and G1 is built. No `Pattern-Departure:` trailer is involved.

One thing outside my lens: `docs/overview.md:1113` still says a check that cannot judge the tests holds the handoff. C1 makes that sentence stale, and the diff leaves it untouched. The affected-set lens owns that.

Tool calls: none. My context already held the record, the diff pieces, the commit log and the cited source.
