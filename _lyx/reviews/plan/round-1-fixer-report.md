# Plan-Review round 1 — fixer report

Every finding was fixed; `lyx loom validate-plan` passes after the changes, with one informational `create-new-unit` finding for card 10's new `internal/burlercli/doc.go`.

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  Premise confirmed: `seatengine.Engine.Probe` stops every live advisor when the chair is not live (`internal/seatengine/resume.go`), and `TestEngine_ProbeRound`'s "found run with its output file on disk stays live" row and `TestEngine_Resume`'s "a round without both halves live is refused and nothing is stopped" row pin the old probe.
  Card 8's "Probe and resume" now records that the seat probe stops a live reviewer beside a gone fixer and `ProbeRound` reports it by its file, so `BurlerProducer` runs a fresh round as it does today.
  The card also states that `BurlerProducer`'s lone-live-reviewer branch stays unreached, and that a probe stop failure is wrapped in `ErrHalfNotStopped` as `Run` wraps one.
  Its test bullet rewrites both rows and adds a probe row for a failed reviewer stop.
- **F2 (LOW), fixed as proposed (both halves of the proposal).**
  Premise confirmed: `ReviewGateEntry` is `PassOnCap` (`internal/burlerengine/verdict.go`), and `GateOutcome.Passed` counts no `PassOnCap` entry (`internal/shuttleengine/gate.go`).
  The decideRound bullet now says no real reviewer reaches the branch and why it stays.
  `TestEngine_Run_RoundFailureRules` gains a row whose fake reviewer's done result carries `Gate.Passed` false and wants the review-gate error with no verdict.
- **F3 (LOW), fixed as proposed.**
  Card 8 names the input that reaches the absent-table refusal: a both-live `LiveRound` built by hand rather than taken from `ProbeRound`, as `shedadapters`' fake-runner tests build one.
  A new `TestEngine_Resume` row wants the refusal with nothing started or stopped.
- **F4 (LOW), fixed as proposed.**
  Premise confirmed: `internal/burlercli` has no `doc.go`, and the decision record's Scope lists burlercli's `doc.go`.
  Card 10 gains `**Create:** internal/burlercli/doc.go`, describing the remaining verbs and the fixer's `lyx shuttle await` wait, and its ImpactSummary says so.
- **F5 (LOW), fixed as proposed.**
  Card 3's Intent gains a Bound paragraph.
  `ForkSubagents` only passes through to the seat's spec, an unset flag authorizes no fork, and the seat engine audits none, so the caller that sets it owns the audit, as Burler's `accept` does.
  The paragraph also notes that an advisor gate adds a check and lets nothing through.
- **F6 (LOW), fixed as proposed.**
  The spec's Card granularity section says removing a symbol's last caller and deleting the symbol are two cards.
  `Shuttle`, `runnerShuttle`, `RunnerShuttle`, its two methods and `shuttle_test.go` move out of card 8 into a new card 12, `delete-burler-shuttle`, inserted with `lyx loom insert-card` after the last card so no later card edits code that still names them.
  Card 8's seam paragraph now says card 12 deletes them.
  Card 8 also drops `shuttle.go` from its header-rewrite list, and its test bullet for `shuttle_test.go` moves to card 12.
  Card 8 keeps its unexported deletions: `Engine.join` calls `decideRound` under its old signature and `Engine.probeHalf` reads the removed `Shuttle` field, so neither compiles past card 8.
- **F7 (NIT), fixed as proposed.**
  Card 8's seam paragraph has `Engine` hold a `*seatengine.Engine` field directly in place of a one-implementer interface.

## Disputed

None.

## Deferred

None.
