---
verdict: BLOCKING
rationale: "The review's F1 to F3 are fixed in the current plan, but cards 3 and 6 each delete a symbol that a test file outside their footprint still uses: card 3 removes loomCLI.armedVerb, read by TestArmAt_RecordsTheArmingVerb in internal/loomcli/wiring_test.go, and card 6 removes shouldReflectFriction, called by TestShouldReflectFriction_AwaitingNeverReflects in internal/loomcli/approve_awaiting_test.go, so go test ./internal/loomcli/... stops compiling after either card"
---

# Plan-Review judge, round 2

The review report judged the plan before the round-2 fixer pass, when it still had eleven cards.
I judged the plan as it stands now, with nine cards.
The report has no `## Focus departures` section, so there are no departures to rule on.

`_lyx/loom/rework/` does not exist, so `_lyx/discussion/decision-record.md` is the whole answer key.

## The review's findings: legitimate, and now resolved

- **F1 and F2 (Tier 1 under `step` and the pending retry):** both were legitimate against the plan the reviewer read.
  The cards that carried them, `anomalies-under-step` and `anomaly-pending-retry`, are gone.
  `tierOneOn`, the Tier 1 stencil rule, the CONSTRAINTS Tier 1 clause and the `selfreport` module line are gone too.
  A search of `_lyx/plan/` for `Tier 1`, `anomal`, `tierOneOn` and `fileIssue` finds only two things.
  One is the overview's new Shared Decision, which states that neither item is carried and that both need the decision record amended through Discussion.
  The other is card 6's `loomPostRun` line, which keeps the existing `detectAndFileAnomalies` call "as today".
  Neither one adds scope.
- **F3 (card 2's ImpactSummary):** it now names `battencli` and `shedcli`.
  I checked both.
  `battencli.ArmAt` is at `internal/battencli/arm.go:413` and `loomcli.ArmAt` at `internal/loomcli/arm.go:266`.
  The claim that the only other `StepEnvelope` caller is one loomcli test also holds: it is `internal/loomcli/step_test.go:41`.
- **Cross-references:** every "card N" mention in the overview and cards 3 to 6 points at the right card after the renumbering.
  Card 6's done-when scenario names `shedfake.Shuttle` as the fake issue filer.

## New, BLOCKING: two removals break the `loomcli` test build

Both are footprint defects grounded in a false claim in the card itself.
The spec treats a card's target list as its declared footprint.
Tier 1 verify runs `go test` on the card's own package, and these cards fail it.

- **Card 3 (`reflect-under-step`)** removes `loomCLI.armedVerb`, its assignment in `armAt`, and every comment that cites it.
  It says the field "has no other reader".
  But `TestArmAt_RecordsTheArmingVerb` in `internal/loomcli/wiring_test.go:681-693` reads `c.armedVerb`, and `wiring_test.go` is not among card 3's targets.
  Card 3's ImpactSummary calls the field "one dead receiver field", which leaves out that test.
- **Card 6 (`halt-notes`)** removes `shouldReflectFriction` "along with its test".
  Its ImpactSummary says the function is one "which only its own test called".
  But `TestShouldReflectFriction_AwaitingNeverReflects` in `internal/loomcli/approve_awaiting_test.go:69-73` calls it too, and `approve_awaiting_test.go` is not among card 6's targets.
  Card 6's own `halt_test.go` covers the property that test pins (`awaiting` reflects nothing).
  So the fix is to remove that test inside card 6, with no need to replace it.

Each fix is small.
Add the test file to the card's `**Edit:**` list.
Say in the Intent that the test is removed and why.
Correct the ImpactSummary's claim about callers or readers.

## Rest of the plan

Every Decision from D1 to D7 and every Constraint is carried, as round 1 found, and the fixer's removals took nothing away from that coverage.
The verify section runs the plain and the integration tests of every package a card targets.
`contracts/recipes` holds no tests, and `plugins/ly/skills/ly-drive/SKILL.md` is pinned by `internal/loomcli/driveskill_test.go`, which the section runs.
Each new seam (`AfterStep`, the `run` error-path extras, `reflectionShuttle`, `noteRefusals`, the halt reflections) states what it lets through and what bounds it.
