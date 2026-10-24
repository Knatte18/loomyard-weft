---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/configreg/keymove.go:26
    summary: An untuned darn.yaml still holding an old default moves into loom.yaml as a tuned value, changing loom's Code-Review hard cap and reviewer model
    fix: In internal/configsync/movekeys.go takeKey, drop a moved darn value, reporting it as dropped, when it equals darn's former template default for that key; restore the darn entries of formerTemplateValues in internal/configsync/configsync.go (darn_reviewer_model sonnet[high], darn_fixer_model "", darn_reviewer_fan "", darn_fixer_fan "", and review_hard_max_bounces 2) and have the move compare against them. Add a row to TestReconcile_MovesKeysAcrossModules in internal/configsync/configsync_test.go where a base darn.yaml holds review_hard_max_bounces 2 and darn_reviewer_model sonnet[high] and loom.yaml keeps code_hard_max_bounces 0 and code_reviewer_model sonnet[medium].
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/loomcli/darnwiring.go:173
    summary: No test pins that commitDarnDiscussion writes the snapshot before committing, or that a snapshot failure leaves nothing committed
    fix: In internal/loomcli/wiring_darn_test.go add a row or assertion that drives commitDarnDiscussion over a hub fixture and checks that _lyx/discussion/board-entry.md is in the discussion commit, and that a missing board entry leaves the discussion directory uncommitted. wiring_test.go:778-783 only asserts that the CommitDiscussion seam is non-nil, so it passes if commitDiscussion replaces commitDarnDiscussion.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/configsync/configsync_test.go:1916
    summary: Six of the nine new darn key moves have no test row, and the default move rule at base and sparse layers is untested for them
    fix: Extend the existing table of TestReconcile_MovesKeysAcrossModules in internal/configsync/configsync_test.go and the TestPrepareRunSettings rows in internal/configreg/runsettings_test.go with rows for review_fan, darn_reviewer_fan, darn_reviewer_model, darn_fixer_model and review_amend_max_bounces, and one row where loom.yaml holds a tuned code_* value at a base layer and one at a sparse layer, asserting the loom value is kept.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/configcli/doc.go:61
    summary: The run-profile example still uses the retired darn key review_max_bounces
    fix: In internal/configcli/doc.go line 61 replace the example {"darn":{"review_max_bounces":2}} with {"loom":{"code_max_bounces":2}}, and move the profile fixtures of internal/configcli/configcli_test.go lines 1167-1187 to the same key.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: scope
    location: internal/loomcli/landingdeps.go:263
    summary: rerouteReviewed and codeBudgetZero now read loom.yaml for a darn run and no test reaches either directly
    fix: In internal/loomcli/landingdeps_test.go (or the file that tests landingDeps) add a table test of rerouteReviewed over a hub with code_max_bounces 0 and above 0, expecting false and true, and with the load of loom.yaml failing, expecting the error returned.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/loomcli/darnwiring.go:186
    summary: The board open and board read refusals of writeBoardEntrySnapshot carry no way forward, unlike the missing-entry refusal
    fix: In writeBoardEntrySnapshot append a way forward to the OpenHub and GetTask error messages, naming the step to take, such as fixing the board and then handing off again, in the form the missing-entry message uses.
    origin: lens:code-pattern-fit
  - id: F7
    severity: NIT
    class: consistency
    location: internal/loomcli/circling_test.go
    summary: The TestPerchHardCap row for the darn recipe's code cap repeats the loom code-cap row's code path
    fix: Delete the TestPerchHardCap row named "a code cap, the darn recipe's perch's too" in internal/loomcli/circling_test.go; the loom code-cap row already covers perchHardCap for Code-Bouncer.
    origin: lens:code-tests
---

### [MEDIUM] Untuned darn defaults migrate into loom.yaml as tuned values

**Location:** internal/configreg/keymove.go:26, internal/configsync/movekeys.go:247, internal/configsync/configsync.go (`formerTemplateValues`)
**Issue:** `ReconcileHubWideAt` applies the key moves first, at the board dir as a base layer.
`takeKey` overwrites a loom value only when it equals loom's own template default, and it never asks whether the darn value was tuned.
A base darn.yaml from before the hub was sparse still holds darn's old defaults, among them `review_hard_max_bounces: 2`.
That value replaces loom's `code_hard_max_bounces: 0`, which means take the run-wide `review_hard_max_bounces` of 4, so loom's Code-Review hard cap drops from 4 to 2.
The diff also removes the `darn` entry of `formerTemplateValues`, so a stale `darn_reviewer_model: sonnet[high]` now counts as tuned and replaces loom's `code_reviewer_model`.
The moved keys are then exempt from the strip pass through `movedTargetKeys`.
The summary says a tuned loom value is kept, which holds, but it says nothing of an untuned darn value.

### [MEDIUM] Snapshot-before-commit ordering has no test

**Location:** internal/loomcli/darnwiring.go:173
**Issue:** The board entry says the darn `CommitDiscussion` seam writes the snapshot before committing.
`TestWriteBoardEntrySnapshot` covers the writer function and `wiring_test.go:778-783` covers only that the seam is set.
Neither fails if `commitDarnDiscussion` commits first, skips the snapshot or is replaced by `commitDiscussion`.

### [LOW] Most new key moves are unpinned

**Location:** internal/configsync/configsync_test.go:1916
**Issue:** One reconcile row covers `review_max_bounces` and `darn_fixer_fan`, and the run-settings rows cover three more.
`review_fan`, `darn_reviewer_fan`, `darn_reviewer_model`, `darn_fixer_model` and `review_amend_max_bounces` have no row, so a mistyped target key passes every test.

### [LOW] Stale retired key in the configcli example

**Location:** internal/configcli/doc.go:61
**Issue:** The diff edited the neighbouring sentence of this doc but left the `JSONEntries` example on `review_max_bounces`, a retired darn key that now moves to `code_max_bounces`.

### [LOW] rerouteReviewed and codeBudgetZero are untested

**Location:** internal/loomcli/landingdeps.go:263
**Issue:** The darn reroute now reads loom.yaml's effective budget through `codeBudgetZero`.
Only the routing tests reach the budget-0 route, and none reaches these two seams.

### [LOW] Snapshot refusals lack a way forward

**Location:** internal/loomcli/darnwiring.go:186
**Issue:** The missing-entry refusal names `lyx board upsert`, but the `OpenHub` and `GetTask` failures return a bare wrapped error, and a halted darn run's parent has no named next step for them.

### [NIT] Redundant perchHardCap row

**Location:** internal/loomcli/circling_test.go
**Issue:** After the darn special case was removed, the darn-recipe row exercises the same path as loom's `Code-Bouncer` row and adds no coverage.

## Rejected

- Static-prose assertion on "Added after Discussion" and "writer's own claim" in `TestDarnPerch_SharesLoomsPerchBarTheAnswerKey` (lens:code-tests): the old darn recipe test already asserted the same substrings, and the text is recipe config, not a stencil, so the diff adds no new assertion.
- Loss of darn rubric rules (focus 1): every rule of `loom-rubric-darn-review` is carried by the shared rubric, the board-entry-only ones under their "with a board entry snapshot" bullets; the range moves to `review_base`, which `Env.ReviewBase` fills on a darn run (`internal/loomcli/wiring.go:688`).
- Leftover `Darn-*` perch names, `NameDarn*` constants or `loom-rubric-darn-review` references (focus 5): only the intended `reportengine` literals, the recipe header's restart note and historical `docs/research` hits remain.
- Budget 0 and hard cap (focus 4): the budget-0 rewrite keys on `Code-Review` and loom's route is unchanged; `perchHardCap` reads loom.yaml for every Bouncer row; verified correct.
