# Card 6 — The plan review rubric points at the target-redundancy checks

**Edit:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/claims_test.go`

**Intent:**
Decision D5, issue #542.
The rubric's "Proposed fixes" section already requires a fix every validation check admits, yet a reviewer still proposed a target pairing the gate refuses.

The "Proposed fixes" section gains one sentence, after the one binding a proposed fix to the validation checks: a fix that adds or pairs targets must pass `redundant-package-target` and `redundant-file-target` in `{{.specs_dir}}/loom/loom-plan-spec.md`, since a file or a package listed beside a member of it on one card is refused, and the spec says what counts and the way forward.
The sentence mirrors the plan template's existing pointer in `loom-template-plan.md` and does not restate the checks beyond that one-clause gist (`PATTERN-producer-pointer-rule`).
One sentence per line.

Coverage: the `loom-rubric-plan-review.md` row of `TestStencilClaims`'s table in `claims_test.go` gains claims scoped to the `## Proposed fixes` section (through `inSection`) for `` `redundant-package-target` `` and `` `redundant-file-target` ``, with a why naming the pointer, as the plan template's row already pins its own.

**ImpactSummary:** one sentence of the rendered plan-review rubric and its claim rows; no Go behavior changes.

**Commit:** 6: loom: plan review rubric names the target-redundancy checks
