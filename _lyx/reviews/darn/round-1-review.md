---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/configcli/doc.go:50
    summary: Stale doc comment still names loom's retired model and fan keys review, fix, star_review, star_fix, star_fan and star_fix_fan
    fix: In the sentence at internal/configcli/doc.go:50 replace the retired names with reviewer_model, fixer_model, each segment's *_reviewer_model and *_fixer_model, and each perch's *_reviewer_fan and *_fixer_fan in loom plus darn's darn_reviewer_fan and darn_fixer_fan.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: design
    location: internal/loomengine/config.go:1087
    summary: copyWebsterModelsOntoMerge copies a retired webster_review or webster_fix value onto Merge even when the document already holds the new webster key, so Merge follows a value the rename then discards
    fix: In copyWebsterModelsOntoMerge skip the copy, or copy from webster_reviewer_model and webster_fixer_model, when the document already holds the new Webster key; add a row for the both-names case to TestMigrateConfig_RetiredKeys.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: internal/darnengine/config_test.go
    summary: The empty-list behaviour of the darn model keys, claimed in the change description, has no TestLoadConfig row
    fix: Add a row with darn_reviewer_model set to an empty list to the existing darn TestLoadConfig table, asserting the load succeeds and the key reads as unset.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/configreg/runsettings_test.go:88
    summary: The run_settings rename is pinned only for fan keys; no row covers a retired model key or the Webster-to-Merge copy
    fix: Add one row to the existing TestPrepareRunSettings table with a retired loom model key such as webster_review, no new top-level test.
    origin: lens:code-tests
---

### [LOW] Stale retired key names in configcli doc

**Location:** internal/configcli/doc.go:50
**Issue:** The doc comment on open-map keys still lists the retired names, although the diff edited this file elsewhere and the board entry lists package docs as in scope.

### [LOW] Merge copy ignores a new Webster key

**Location:** internal/loomengine/config.go:1087
**Issue:** With both `webster_review` and `webster_reviewer_model` present, Merge gets the retired value while Webster keeps the new one. Rare input, but it contradicts the rule that the new key wins.

### [LOW] Darn empty-list behaviour untested

**Location:** internal/darnengine/config_test.go
**Issue:** `LoadConfig` skips validation for an unset darn model key, including `[]`, and the change description says so, but no row pins it.

### [LOW] run_settings rename coverage covers fan keys only

**Location:** internal/configreg/runsettings_test.go:88
**Issue:** The hook runs on run_settings generically, so the risk is small, but the model-key renames and the Merge copy have no row there.

## Rejected

- Darn error messages carry the prefix "loom: ResolveReview:" (lens:code-correctness, NIT): the key and file named are correct, and the prefix names the resolver function.
- `yamlengine.RenameKeys` has no direct test (lens:code-tests, LOW): indirect coverage through both Migrate tests is acceptable under `PATTERN-test-economy`.
- Leftover old names: a repo-wide search found them only in migration tables, migrate comments and `docs/overview.md:545`, which describes the retired names on purpose.
- Repository rules: the test-budget row for `darnengine` rises to match its one new top-level test, `doc.go` and `docs/overview.md` landed in the same diff, and no `PATTERN` entry is broken.
- The change description's acknowledged unmet criterion (a strict load does not refuse old keys) is recorded openly and the tests pin the reconcile rename instead; not raised as a defect of the diff.
