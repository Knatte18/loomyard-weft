# Plan-Review round 3 — fixer report

## Fixed

### F1 — card 7's Call test conflicts with the gate-once fake from card 6

Validated: `TestDiscussionWriteEntry_ParentReviewBeginRound`'s `AttachDoesNotBeginRound` subtest scripts a Died attach because the fake shuttle's gate evaluation on a done run opens a parent-review round, and `newTestEnv`'s support log is absent (`TestDiscussionWriteEntry_GateConfig` sees "support log does not exist").
Card 6's fake would do the same on card 7's gated done chairs.

- Card 7: `TestDiscussionSeatsEntry_Call` now states that every done chair result it scripts, through `Results`, `RunFn` or `ResumeFn`, carries a passing `GateOutcome`, so the fake never evaluates the row's gate, and says why.
- Card 6: a sentence requires a later card's test over a gated table with a done chair to state whether it carries a `GateOutcome` or relies on the fake's evaluation; the ImpactSummary now says "today" and names card 7's test as carrying the outcome and card 8's row test as relying on the evaluation.

### F2 — the row substitution bypasses the gate-quiescence guard with no stated bound

Validated: `gatedRowFanViolations` in `internal/loomrecipe/gatequiescence_test.go` admits gated rows only for `BurlerRound` and the four writer engines, and `TestNoGatedRowAuthorizesForkSubagents` reads only `shedbuild.Parse(recipes.LoomRecipe)`.
`internal/seatengine` never sets `ForkSubagents`: its one `shuttleengine.Spec{` literal is in `prompt.go`.

- Card 8: `Edit:` gains `internal/loomrecipe#gatedRowFanViolations` and `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`.
- Card 8 Intent: a `Bound:` sentence names what the substituted row escapes and what keeps the chair fork-free, and a sentence adds `DiscussionSeats` to the guard's writer case, message and doc, plus a test row running the guard over the shipped recipe after `applyDiscussionProducer(recipe, true)` with the review fans, expecting no violation.
  It is a row in an existing test, so the test budget is unchanged.
- Card 8 ImpactSummary names the guard.
- Overview's `two-producers-selected-by-config` rationale now names the gate-quiescence guard beside the coverage guard.

`lyx loom validate-plan` passes, with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
