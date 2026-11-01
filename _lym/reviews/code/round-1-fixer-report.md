# Fixer report — round 1

## Changes

- F1 (fixed as proposed, commit 89c85b32): `seam-test-unlisted` in `contracts/specs/loom-plan-checks-spec.md` now states the package-counterpart coverage rule the gate applies.
- F2 (fixed as proposed, commit 89c85b32): one clause per entry in `contracts/specs/loom-plan-checks-spec.md` for what the gate does not see, in `caller-uncovered`, `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted`, taken from the base spec's Bound notes.
- F3 (fixed as proposed, commit 504b4052): `contracts/stencils/loom/loom-rubric-plan-review.md` says `ImpactSummary` is required for `Edit`, `Extends` and `Delete`.
- F4 (fixed as proposed, commit 2f1fe1a3): `contracts/stencils/webster/webster-template-plan-fix.md` points the fixer at `loom-plan-csharp-spec.md` for a `csharp` plan.
- Self-check follow-up (below): `contracts/stencils/loom/loom-block-plan-cards.md` placeholder now reads "Edit/Extends/Delete targets".

## Self-check

- fix-affected-set, LOW: `loom-block-plan-cards.md:113` omits `Extends` — fixed as proposed.
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
