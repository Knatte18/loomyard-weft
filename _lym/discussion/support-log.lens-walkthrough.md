```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, with the end state checked against the decision's own purpose.

**Findings**

1. MEDIUM — D5 (also Constraints and Acceptance criteria): reclassifying the key also re-admits the retired darn keys, and the record doesn't say so.
   - Case: a task's `run_settings` holds `darn.darn_reviewer_fan: ""` or `darn.review_fan: ""`.
   - Today `internal/configreg/keymove.go` moves each to `loom.code_reviewer_fan`, and `runsettings_test.go` (lines 195–204) pins that the moved key "is refused as a guard key".
   - After D5 the moved key passes `ValidateLocal`, so both retired spellings start working as live settings.
   - The purpose still holds, but the record should state this as part of D5's bound.
   - Pinned tests that flip with D5: `configreg_test.go:86` (loom's local-denied list) and `runsettings_test.go:195-204`. The plan finds these itself; the record should still say the retired-key route opens too.

2. MEDIUM — PATTERN changes and Notes: the `changes:` bullet repeats the current overview text unchanged.
   - Code-Review's "Declared PATTERN changes landed" check reads PATTERN.md's diff for each declared entry.
   - `PATTERN.md` gets no diff for this entry, since only `pattern/PATTERN-config-layers.md` changes.
   - A reviewer may then raise BLOCKING "declared change not landed", or the check passes on text that never changed.
   - Fix: drop the bullet and keep the background edit in D7, or change the overview text itself.

3. LOW — D6: the acceptance line covers only a seeded pair's load. No line has a light-profile Code-Review round read `[""]` and pick the solo cluster stencil (`burler-cluster-solo.md`) over the fan.
   - The `FanList` grammar makes this follow; the end state matches the purpose.

**Per decision: case used, purpose met or not**

- **D1** — Case: twill fork list from `discussion_lenses: twill-discussion` after the edit. It yields 6 lenses with `board-scope`. `review-discussion` keeps every lens inside the writer fan, so the writer-covers-reviewer check holds. The dormant `discussion` fan still resolves. Met.
- **D2** — Case: `review-plan` resolve gives `affected-set`, `guard-reach`, `repo-invariants`. The `absorbedBy` loop in `config_test.go` fails on `repo-invariants` unless the reviewer-only exception is added; the record states that. Met.
- **D3** — Case: `lens_set: csharp` with the layer fan `[code-correctness, code-affected-set, code-tests, numerics]`. The base-first, append-only layer rule holds against the new 3-lens base. Met.
- **D4** — Case: a diff adding a new branch with no test gives "none" in the list. A diff adding a redundant test is no longer this lens's finding; the lead's rubric "Test economy" still covers it. Met.
- **D5** — Case: profile `light` darn block `{loom: {code_reviewer_fan: ""}}`.
  - `checkProfiles` → `checkLocalDocument` → `ValidateLocal` is admitted once the category is None.
  - `loomengine.ValidateLocalValues` (`sheddriver.CheckLocalOverride` and `reviewbudget.CheckLocal`) doesn't inspect fan keys, so it passes.
  - Met, apart from finding 1.
- **D6** — Case: Seed-Child for a darn task on `light`. The pair's local `loom.local.yaml` holds `code_reviewer_fan: ""`, which `FanList` decodes as `[""]`, so round 1 runs solo. `solid` gives `["review-code", ""]`, so the fan runs. Met for new hubs. The live hub file is in Open risks.
- **D7** — Case: the `docs/overview.md` line "all six closed to the local layer" becomes false after D5; D7 rewrites it. Met.

**Tool calls**

- `grep ValidateLocalValues` and `LocalDenied` in loomengine and configengine: to check that D5's value-check step admits a fan key. My context lacked the value-check body.
- `grep CheckLocalOverride` and every `code_reviewer_fan` use outside config.go: to trace what else consumes the reclassified key. My context lacked the keymove tests and pinned denied lists.
