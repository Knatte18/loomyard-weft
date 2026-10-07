---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-hold-turn-end.md
    summary: Card 5 names the tests to rewrite by enumeration and misses four tests that still expect an asking outcome from Wait or pollEventsTick, one of them in attach_test.go which is not among its targets
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/13-tail-in-stencils-and-steer.md
    summary: Card 13 rewrites only the add nothing to it sentence of the driver stencil, while the sentences directly above it still say nothing more and verbatim, which the added tail contradicts
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/14-replay-corpus.md
    summary: Card 14 bundles the new public Clock and SetClock seam, a rename across five shuttleengine files, into the corpus card and states no bound on what the seam lets a caller skip
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/05-hold-turn-end.md
    summary: The ImpactSummary pins a tally of five packages for the AwaitOperator assignments, but the card edits assignments in three packages besides shuttleengine, so the number is wrong and perishable
---

## Focus verdicts

1. The unexported `clock` interface renamed to `Clock` (card 14): not a defect.
   Every in-package file that names the type is among the card's Edit targets: `wait.go` (declaration), `run.go` (two struct fields), `fixture_test.go`, `wait_test.go` and `startup_test.go` (the interface assertions and the fixture fields).
   The other test files in the package (`attach_test.go`, `gate_pending_test.go`) only set the struct field or build fakes and never name the type, so `go test` of the package still compiles.
2. The hold and the notifier seam (cards 5 and 6): not a defect.
   The hold (card 5) and the notifier with `StrandName` and `NotifiedOffset` (card 6) are separate cards, and every card cross-reference and the Card Index agree.
   Card 5 states that the offset is the one just past the turn end's line, computed once in `pollEventsTick` and kept by `recordWaiting`.
   `expiredTurnEnd` reports that offset on a later tick, and card 6 repeats it for the expired-shell hold.
   Card 6 states what a failed `NotifiedOffset` write does: it logs Warn, skips that one notice, ends nothing and adds no return site to Wait.
   A plain held Stop is reported once and the expired-shell path clears `waitingTasks` after reporting, so a skipped notice cannot repeat on later ticks.
3. The shed driver stencil statement (card 13): a defect, F2.
   The placement is stated (`## Notifying the parent`), but the card does not reconcile every sentence of that section.

### [MEDIUM] Card 5's test list misses tests that break when Wait stops returning asking

**Location:** `_lyx/plan/05-hold-turn-end.md`, the Tests paragraph.
**Issue:**
The card rewrites "the existing tests that expect `OutcomeAsking` from Wait" and names five.
Four more still expect it and fail from card 5's commit on, since the constant survives until card 12 and nothing else changes their expectation:
`TestRun_Wait_EventsHandling` and `TestRun_Wait_Finalize_OutcomeWriteFailure_StillReturnsClassifiedResult` in `wait_test.go` (a Stop with the output file missing, expecting `OutcomeAsking`);
`TestPollEventsTick_LaterTurnEndListingExpiredShellEndsWithoutNewWait` in `waiting_test.go`, which asserts `OutcomeAsking` from `pollEventsTick`;
and the `BacklogEndsInAsk_ClassifiesAsking` subtest of `TestAttach_OffsetStartsAtZero` in `attach_test.go`, a `running` record whose Wait would now run to the deadline.
`attach_test.go` is not among card 5's targets, and card 7 and card 12 mention it only for asking candidates and leftover constant uses, so no card owns that subtest at the commit where it breaks.
The enumeration misses a class of tests, not one name.
**Fix:**
State the rule in the card instead of a list: every existing test that expects `OutcomeAsking` from `Wait` or `pollEventsTick` is rewritten to the held reading, or to `OutcomeTimeout`, `OutcomeDied` or `OutcomeDone` where it only needs some other outcome.
Name the four missing tests, and add `internal/shuttleengine/attach_test.go` to card 5's Edit targets.

### [MEDIUM] Card 13 leaves the driver stencil's nothing more and verbatim contradicting the tail

**Location:** `_lyx/plan/13-tail-in-stencils-and-steer.md`, the shed driver stencil bullet.
**Issue:**
In `contracts/stencils/shed/shed-template-driver.md`, `## Notifying the parent` says the generic escalation is one short SendMessage naming the run-id and the stop report path "and nothing more", and that a `parent_notice` is sent "verbatim instead of this generic line".
The card changes only the sentence "The notice is the whole message: add nothing to it".
After that edit the section would still say "nothing more" and "verbatim" two sentences before permitting the tail, which is the contradiction with the existing verbatim rule that the focus directive asks about.
The section-level statement at `## Stopping on a non-running state` ("relayed to the parent") is consistent and needs no change.
**Fix:**
Name all three sentences in the card: the generic line becomes "naming the run-id and the stop report's path and nothing more, then the tail", the `parent_notice` sentence becomes "verbatim, followed by the tail", and the whole-message sentence becomes "add nothing to it but that tail".
The claims in `claims_test.go` then pin the one rule the section states.

### [LOW] Card 14 bundles the public clock seam into the corpus card without a bound

**Location:** `_lyx/plan/14-replay-corpus.md`
**Issue:**
The exported `Clock` type and `Runner.SetClock` are a new public seam of `shuttleengine`, used from another package (`claudeengine`'s corpus test).
They are a separately reviewable unit: a rename touching five files plus a new method.
The card folds them into the corpus card, the largest in the plan, with 21 created data files and one test, and declares neither a `Create` target for `SetClock` nor a card others could `Uses`.
The card also adds an escape hatch, a caller that can replace the time source of a run, and its Intent bounds it only with "production wiring never calls it", which nothing enforces.
**Fix:**
Move the `clock` to `Clock` rename and `SetClock` into their own card ahead of the corpus card, with a `Create` handle for `Runner.SetClock`, and have card 14 `Uses` it.
In the Intent, state what the seam can skip (real sleeping and the real deadline) and what bounds it (only a test or replay caller sets it, and `NewRunner` and `NewDetachedRunner` default to the real clock).

### [NIT] Card 5's ImpactSummary pins a wrong tally

**Location:** `_lyx/plan/05-hold-turn-end.md`, ImpactSummary.
**Issue:**
"Spec.AwaitOperator is gone with its assignments in five packages" is a count that goes stale and is wrong here: the card edits assignments and assertions in `orchcli`, `loomengine` and `loomcli` besides `shuttleengine`, which is four.
The fifth package that names the field, `shedadapters`, only has a comment, edited by card 9.
**Fix:**
Write "gone with its assignments in the callers' packages" and drop the number.
