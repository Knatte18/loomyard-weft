# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `internal/shuttleengine/attach_test.go`, `TestAttach_RemovesSupersededStrands`.
  Added a `gatedIfLive` field and the row `attach_if_live_gated_removes_nothing`, which attaches through `runner.AttachIfLiveGated(spec, GateSpec{})`.
  The commits are `67403a42` and `64cd4aae`; `lyx gate test ./internal/shuttleengine` passed.

## Disputed

None.

## Deferred

- F2: no sheddriver test reaches the shuttleengine runner fixtures.
  `newFixture`, `fakeReed` and `fakeEngine` are unexported test code in `package shuttleengine`, so a `RunnerShuttle` test cannot build a runner cheaply.
  The review's fix key says to leave the gap and accept it in that case, so no change was made.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the test's doc comment named only `AttachIfLive` as removing nothing. Fixed as proposed in `64cd4aae`.
- fix-proposal: no findings.
- fix-affected-set: no findings; the two entry points that pass `removeSuperseded=false` are both pinned.
