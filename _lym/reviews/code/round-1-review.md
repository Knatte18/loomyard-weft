---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configsync/configsync_test.go:1543
    summary: 'the new former-value strip of active cautious has no test row, so deleting the formerTemplateValues entry leaves every test green'
    fix: 'In TestReconcileAll_MigratesToSparse add a batcher row whose worktree file holds only the line active: "cautious", with migrate options and an unmarked board, expecting wantStripped to be [active], the file deleted, and wantMarker true. Add a batcher row on a marked board (marked true) whose file holds the same line and expects wantFile unchanged, so the pin is covered. Reuse the existing row struct; add no new test function.'
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: tools/sandbox/SANDBOX-WEBSTER-SUITE.md:5
    summary: the sandbox suite doc still names cautious as the default batcher profile
    fix: 'In line 5, change the words the `cautious` cost profile by default to the `wide` cost profile by default.'
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/batcher/config_test.go:406
    summary: 'nothing pins that wide differs from cautious only by budget; the shared-weights test compares Weights alone, so max_cards or the budgets can drift unseen'
    fix: 'In the templateProfilesShareTheMeasuredWeights subtest, also read the max_cards of each profile and budget (through the batcher accessors the package already exposes, or by building each cost batcher with the inputs of NewCost from Active) and assert the wide budget is 300000, the cautious budget is 200000, and max_cards is equal. If no accessor exists, leave this out and drop the finding rather than adding one.'
---

### [MEDIUM] Former-default strip of `active: "cautious"` is untested

**Location:** `internal/configsync/configsync_test.go:1543`

**Issue:** The commit adds `formerTemplateValues["batcher"]["active"] = "cautious"` (`internal/configsync/configsync.go:52`) so an unmigrated hub moves to `wide`. The only batcher row in `TestReconcileAll_MigratesToSparse` uses `batcherDiffering`, the current template with one orientation changed. There `active` is stripped as equal to the new template's `wide`, not as the former value. `TestFormerTemplateValues_DifferFromTheTemplate` checks only that the former value differs from the template.

### [LOW] Sandbox doc still names `cautious` as the default

**Location:** `tools/sandbox/SANDBOX-WEBSTER-SUITE.md:5`

**Issue:** The decision record asks that comments naming `cautious` as the template default follow "anywhere else that says so". This line says "the `cautious` cost profile by default". A grep of the repository for `cautious`, outside tests and research docs, finds no other place that calls it the default.

### [LOW] Nothing pins that `wide` is `cautious` with only the budget changed

**Location:** `internal/batcher/config_test.go:406`

**Issue:** The decision record says `wide` is `cautious` with `budget: 300000` and no estimator weight changes. `templateProfilesShareTheMeasuredWeights` compares the two `Weights` values, but `max_cards`, `batchifier` and each budget value are asserted nowhere. `nonPositiveBudgetErrors` edits the literal `budget: 300000`, so it fails only when that text is absent. I read `template.yaml` and the two profiles match as the record asks; this is about the pin, not the current content.

## Rejected

- Code-correctness F1 (the strip also moves a hub that chose `cautious` on purpose, undocumented in the comment): the strip runs only on a board without the base-sparse marker (`unmarked` at `internal/configsync/configsync.go:985`, `os.IsNotExist(markerStat)` at `:948`). Such a board holds a full template copy, where a deliberate `cautious` is indistinguishable from the old fill. A marked hub keeps `active: "cautious"` as a pin. The commit message states the effect, so the behavior is acknowledged and unavoidable, and the comment states the mechanism ("an older template generation's value is kept as tuned").
- Code-affected-set note on the `Co-Authored-By: Claude Opus 5.5` trailer: not a finding; the rubric covers commit messages only for `Pattern-Departure:` trailers.
- Handler checks that found nothing: `template.yaml` has `active: "wide"`, `wide` equal to `cautious` bar `budget: 300000`, and `cautious` and `identity` unchanged with the same weights. No estimator weight changed. `config.go`, `registry.go`, `docs/overview.md`, `internal/batcher/doc.go` and `internal/websterengine/doc.go` follow the new default. `templateOrientation` still finds the `cautious` profile it reads. `PATTERN.md` and `pattern/` carry no diff, and the directive declares none.
