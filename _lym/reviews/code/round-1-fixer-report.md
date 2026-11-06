# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `contracts/stencils/webster/webster-template-master.md` now ends the `plan_order` sentence with a full stop and says a `plan_order` refusal skips the fix-plan rung: the master writes `outcome: stuck` with the refusal quoted verbatim. The numbered fix-plan steps now sit under "For `plan_defect` and `tree_moved`:". Commit a5fdc943.
- F2 (fixed as proposed): the same stencil no longer quotes one way-forward wording; it says the later card's `begin-batch` refuses with a `plan_conflict` refusal that names the way forward. Commit a5fdc943.
- F3 (fixed as proposed): `internal/planglyph/deleteorder_test.go` and `internal/planglyph/planfixture_helper_test.go` import `internal/planindex` and compare `f.Check` to `planindex.CheckDeleteBeforeReference`. No test added. `go vet ./internal/planglyph` is clean. Commit 545ea755.
- Self-check follow-ups (see below): the master stencil heading and the "most often" sentence, the plan-fixer section of `internal/websterengine/doc.go`, and the plan-fix stencil's header comment now state that `plan_order` ends stuck without a plan fixer. `go vet ./internal/websterengine` is clean.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: the master stencil heading and line 239 still said every plan conflict goes to the plan fixer. Fixed as proposed.
- fix-proposal, LOW: `internal/websterengine/doc.go` plan-fixer section still sent every `plan_conflict` to `fix-plan`. Fixed.
- fix-proposal, LOW: the plan-fix stencil header comment said it is spawned after any `plan_conflict`. Fixed.

## Disputed

None.

## Deferred

None.
