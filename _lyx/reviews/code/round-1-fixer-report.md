## Changes

All four findings fixed as proposed, one commit each.

- F1: a `decisionReview` row in `TestBurlerFixProducer_Call_ProbeAndResume` (`internal/shedadapters/burlerfix_test.go`) gives a live fixer, a told notifier and a review holding a decision-class finding; it asserts the fixer was resumed, not started, and no line was posted.
- F2: `TestBurlerFixEntry_RoutesDecisionFindingsThroughEnvNotifyParent` (`internal/shedrecipe/entries_burlerhalves_test.go`) builds a BurlerFix row under an `Env.NotifyParent`, calls it over a decision-class review, and asserts one line posted and `DecisionsRouted` set. The wiring closure in `internal/loomcli/wiring.go` is left uncovered.
- F3: removed the findings-notice bullet from `contracts/stencils/orch/orch-template-role.md`.
- F4: `webster-body-implementer.md` and `webster-body-verify-fix.md` now point at the notice the parent directive admits instead of quoting its opening.

`lyx gate test` over `internal/loomcli`, `internal/websterengine`, `internal/shedadapters` and `internal/shedrecipe` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings. It noted it had not confirmed that every role reading the two bodies holds the parent directive; fix-proposal confirmed it (the cold verify-fix and recovery prefixes and the Master stencil render `{{.parent_directive}}`).
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
