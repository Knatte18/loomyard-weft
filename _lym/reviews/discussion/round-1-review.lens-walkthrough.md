```yaml
lens: walkthrough
outcome: reported
```

Task: walk each decision's mechanism through the walkthrough lens.

**List**

- D1: `board-scope` is added to `twill-discussion` and the dormant `discussion` fan, and its prose ends in its own list. Then `TestConfig`'s lens checks run. **Purpose not met:** the emphasis-line exemption is computed only from `review-*`/`fix-*` fans, so the test requires `board-scope` to end with the emphasis line, and D1's "Ends in its list" contradicts that.
- D2: `review-plan` becomes `affected-set`, `guard-reach`, `repo-invariants`. The writer-covers-reviewer loop runs with `absorbedBy`, and `repo-invariants` lies in a `review-` fan, so it is exempt from the emphasis line. **Met**, assuming the check gains the reviewer-only exemption the acceptance line names.
- D3: base `review-code` has 3 lenses and the `csharp` layer resolves to 4 with `numerics`. `code-affected-set` prose gains the rule documents and the four rubric names. **Met.**
- D4: `code-tests` prose loses the "Test economy" sentence and the open-existing-tests step, and keeps its per-behaviour list. **Met.**
- D5: a `light` darn block `loom: {code_reviewer_fan: ""}` goes through `PrepareRunSettings`, then loom `MigrateConfig`, then `checkLocalDocument` (`ValidateLocal` now passes under `CategoryNone`; `ValidateLocalValues` doesn't check fans). Seed-Child then writes the pair's `loom` local file. `LoadConfig` gets `FanList{""}`, and `ValidateFanKeys` skips the empty entry. `SettingLookup` returns the snapshot value `""`, so `fanSetting` returns a nil list and the review runs solo. **Met.**
- D6: a fresh hub seeds the template `profiles.yaml`, and a darn task with the default `light` profile goes through `LayerRunSettings`. Its local loom file holds `code_reviewer_fan: ""`. `ReconcileAll` then runs `reconcileSparseFile` with a nil strip, and `Prune` keeps the key because it is a template key. The Code-Review Bouncer and BurlerReview read a nil fan, so the round is solo. **Met on a newly seeded hub.** On this hub the change doesn't reach the pair, as Open risks states.
- D7: walking the named sites for text that calls the fan keys closed:
  - `docs/overview.md:583` says "all six closed": present.
  - The `loom` `ConfigSpec` comment: present.
  - The `internal/loomengine/template.yaml` key comment and `internal/configcli/doc.go`: no such text. **Partly not met:** two named sites have nothing to change.

**Findings**

1. **MEDIUM, design.** D1 / Acceptance criteria (D1) / Constraints ("procedure lenses end in their own list").
   `board-scope` sits in no `review-*` or `fix-*` fan, so the template test does not treat it as a procedure lens and fails unless its prose ends with the emphasis line "Report anything else you notice too — emphasis, never exclusion."
   D1's "Ends in its list" and the Constraint therefore produce a lens the existing test refuses, and the record states neither the emphasis ending nor a change to the test's exemption rule.
   Evidence: `internal/lenses/config_test.go:298` derives `procedureLenses` from fans prefixed `review-`/`fix-`, and `:341-343` requires every other non-`generic` lens to end with the emphasis line. `goal-scope` and `board-fit` are exempt today only because they sit in `review-discussion` (`internal/lenses/template.yaml:404-410`), which D1 removes. `leanness` (template.yaml:132-137), the other twill-only lens, ends with the emphasis line.

2. **LOW, consistency.** D7, second bullet.
   D7 names loom's `template.yaml` key comment and `internal/configcli/doc.go` as text that "calls all reviewer fan keys closed to the local layer", but neither holds such text. The bullet hands the plan writer two edit sites that don't exist.
   Evidence: `internal/loomengine/template.yaml:15-25`: the fan key comments never mention the local layer, and a grep for `local` in the file found nothing. In `internal/configcli/doc.go`, a grep for `closed|denied` found nothing, and line 56 mentions the fan keys only for value shape.

Tool calls: none.
