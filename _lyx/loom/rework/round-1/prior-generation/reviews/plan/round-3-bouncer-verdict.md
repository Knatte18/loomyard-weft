---
verdict: APPROVED
rationale: "All four round-3 findings are fixed in the current plan: card 3 lists internal/loomcli/wiring_test.go and removes TestArmAt_RecordsTheArmingVerb with loomCLI.armedVerb, card 6 lists internal/loomcli/approve_awaiting_test.go, removes both shouldReflectFriction tests and moves the RunAwaiting property into halt_test.go's loomPostRun bullet, verify's untagged line runs ./cmd/lyx/..., and card 3 renames the armed-for-run test; a repo grep finds no other reference to a removed symbol outside its card's targets, and D1 to D7 stay covered"
---

# Plan-Review judge, round 3

The review report judged the plan before the round-3 fixer pass.
I judged the plan as it stands now.
The report has no `## Focus departures` section, so there are no departures to rule on.

`_lyx/loom/rework/` does not exist, so `_lyx/discussion/decision-record.md` is the whole answer key.

## The review's findings: legitimate, and now resolved

- **F1 (card 3, `armedVerb`):** legitimate.
  `grep -rn armedVerb` finds the field only in `run.go`, `arm.go`, `cli.go`, `wiring.go`, `friction_test.go` and `wiring_test.go`, and card 3 now lists every one of them.
  Its Intent names `reflectFrictionRow`'s condition as the only production reader, removes `TestArmAt_RecordsTheArmingVerb` with the field, and the ImpactSummary names that test.
- **F2 (card 6, `shouldReflectFriction`):** legitimate.
  The function's references are `run.go`, `arm.go:370`, `friction_test.go` and `approve_awaiting_test.go:69-73`, and card 6 now lists all four.
  Both tests are removed, the `RunAwaiting` property moves to `halt_test.go`'s `loomPostRun` bullet, and the ImpactSummary names `loomPostRun` as the production caller.
  That added test case is licensed: it keeps an existing guarantee, it adds no scope.
- **F3 (verify):** legitimate.
  The untagged `go test` line now runs `./cmd/lyx/...`, so `TestLyDriveSkill_IsRecipeBlind` runs against card 7's rewrite.
- **F4 (test name):** legitimate.
  Card 3 renames the test to `TestReflectFrictionRow_ReflectsWhenTierTwoOn`.

## The focus's third question: no other footprint gap

I grepped the other symbols the cards remove or re-sign.
`archiveFrictionDir` and `deleteStaleReport` live only in `internal/frictionengine/reflect.go`, which card 1 edits.
`StepEnvelope` is called only from `shedverbs/step.go`, `shedverbs/step_test.go` and `loomcli/step_test.go:41`, and `stepErrFields` only from `step.go`; card 2 lists all three files.
`frictionengine.Deps` is built only in `loomcli/run.go:82` and `reflect_test.go`, both in card 8, so the new required `TaskSlug` breaks no other caller.
`frictionengine.Shuttle`'s one production implementer is `*shuttleengine.Runner`, which already has `Attach`.
`shedfake.Shuttle` already has `DuringRun` and `DuringAttach`, which card 1's tests rely on.
`newRelativeFrictionCLI`, `TestReflectFrictionRow_SkipsWhenArmedForStep` and `TestStepEnvelope_KeySetIsExactlySixteen` each sit only in a file their card edits.

## One non-blocking remnant

`internal/loomshed/frictionreflect.go:19` (the `frictionReflect` doc comment) still lists "armed for step" as a reason the closure might not reflect.
Card 3 removes that gate and fixes the matching premise in `interruptpolicy.go` and in the recipe comment, but `frictionreflect.go` is not in its targets.
The comment goes stale but no build or test breaks, so I do not block on it.
The implementer can drop the clause while editing card 3, since the comment describes the gate that card removes.

## Rest of the plan

Each of D1 to D7 is carried: D1 by card 3, D2 by cards 2 and 6, D3 by cards 4 and 5, D4 by card 7, D5 by card 1, D6 by card 8, D7 by card 9.
The overview's Shared Decision keeps the board's post-Discussion Tier 1 additions out, as the decision record requires.
The verify section runs the plain and the integration tests of every Go package a card targets.
`contracts/recipes` and `plugins/ly/skills/ly-drive` hold no tests of their own; their pins run under `./internal/loomcli/...` and `./cmd/lyx/...`.
Each new seam states what it lets through and what bounds it.
