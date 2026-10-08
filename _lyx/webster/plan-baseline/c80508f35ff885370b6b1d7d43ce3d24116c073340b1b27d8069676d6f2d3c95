# Card 6 — plan-review-fix-rule

**Prosa:**
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:** Make the Plan-Review reviewer propose only fixes the plan gate admits (D9, the reviewer half of issue #460), and tell the plan writer the nested-module form the reviewer and gate now hold it to (D10).

Rubric, in a new `## Proposed fixes` section after `## Also flag`: a finding whose fix adds, drops, moves or pairs targets, or changes a verify command, proposes only a fix that every check in `{{.specs_dir}}/loom/loom-plan-spec.md`'s validation-checks section admits.
When the reviewer sees no admissible fix, it still reports the defect and names the check that constrains the fix.
The `**Fix:**` line is a proposal: the fixer may apply any other fix the checks admit, and when none exists it defers the finding with the constraining check as the reason, under burler's existing fix rule.
The section points at the spec section and copies no check list (`PATTERN-producer-pointer-rule`).
Bound, in one sentence there: the rule narrows only which fix shapes a reviewer proposes; no defect becomes unreportable.
The rubric's verify-coverage bullet adds that a nested module's packages count as covered by a command run inside that module (`go -C <module>`).

Template: the verify guidance beside "must cover every package any card targets" adds that a nested module's packages are tested with `go -C <module>` and module-relative package paths, and points at `verify-nested-module` in the spec rather than restating it.

`claims_test.go` gains claim rows: for the rubric, the binding of a proposed fix to the validation-checks section, the reviewer naming the constraining check when no admissible fix exists, and `go -C` counting as coverage; for the template, the `go -C <module>` form and the `verify-nested-module` pointer.
The existing claim rows keep passing with the new wording.

**ImpactSummary:** two loom stencils and their claim rows; every Plan-Review and Plan-Write session reads the new text on the next deploy, and no Go behavior changes.

**Commit:** 6: Plan-Review proposes only admissible fixes; the plan template states the go -C form
