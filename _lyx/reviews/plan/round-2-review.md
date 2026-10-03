---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/10-package-local-scans.md
    summary: internal/gitrepo/noforceadd_test.go, the Never Force-Add Invariant scan, is a target of no card, so D2's every-invariant-scan conversion is incomplete
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/15-recipe-row-table.md
    summary: card 15 removes loomRowEngines, but the production doc comment at internal/loomrecipe/names.go:16-18 names it and is not a target
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/11-cli-tree-walk.md
    summary: card 11 deletes drift_test.go and longlist_test.go, but cmd/lyx/seamsignature_test.go:7 and cmd/lyx/sandbox_coverage_test.go:39 name them and are not card 11 targets
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/28-statuscommit-seam.md
    summary: card 28 moves the deps struct, but loomcli's commitStatusDeps and newCommitStatusSeam(deps) are also used by untargeted commitrecords.go, commitrecords_test.go and the protected integration test, and the card does not say they stay
  - id: F5
    severity: LOW
    location: _lyx/plan/27-shedverbs-duplicates.md
    summary: card 27 deletes loomcli's TestStepEnvelope_KeySetIsExactlySixteen, whose shedverbs twin's comment at internal/shedverbs/step_test.go:24 says it mirrors the loomcli test; the intent does not say to rewrite that comment
---

### [BLOCKING] The Never Force-Add scan is converted by no card

**Location:** `_lyx/plan/10-package-local-scans.md` (and the plan's card set as a whole)

**Issue:** `internal/gitrepo/noforceadd_test.go` machine-checks `CONSTRAINTS.md`'s Never Force-Add Invariant (its file comment says so) by walking source: `os.Getwd()` plus `os.ReadDir` over its own package directory, a banned-token set (`"-f"`, `hasPathspecMagic`) and a floor (`noForceAddMinScannedFiles = 5`).
D2 binds every test that enforces a `CONSTRAINTS.md` invariant by walking source, and the acceptance criterion requires every such scan on `scankit`.
No card targets the file: a grep of `_lyx/plan/` for `noforceadd` finds nothing, and card 7 touches only `internal/gitrepo/internal/gitoracle/leaf_enforcement_test.go`.
The focus directive's sweep found no other gap: every other `_test.go` file that reads source (`os.Getwd`, `os.ReadDir`, `filepath.Glob`, `os.ReadFile` of a `.go` file, `go/parser`) and cites a `CONSTRAINTS.md` invariant is already a target of cards 2–10.
The remaining source readers cite no invariant: `fabricengine/mergepaths_test.go` (one-file wiring pin), `burlercli`/`webstercli` `wiring_test.go`'s watchdog scans (a batch scope note) and `shuttlecli/modelpin_test.go` (an R2-F3 regression guard).
The other hits read runtime fixture directories, not source.

**Fix:** Add `internal/gitrepo/noforceadd_test.go` to card 10's `Edit` list.
Its intent gains the Never Force-Add scan, converted to a shallow `scankit` walk of its own package directory with production-file filter, keeping its banned tokens and its floor of 5 verbatim, and proved by one planted `"-f"`.

### [MEDIUM] A production doc comment still names `loomRowEngines`

**Location:** `_lyx/plan/15-recipe-row-table.md`; `internal/loomrecipe/names.go:16-18`

**Issue:** Card 15 folds `coverage_guard_test.go`'s `loomRowEngines` into `wantProducerTable` and moves its readers onto that table, so the variable goes away.
`RecipeEngines`' doc comment in `names.go` justifies itself against "a second hand-maintained table alongside loomRowEngines", which then names nothing.
`names.go` is not a target of card 15, and the card's ImpactSummary says `internal/loomrecipe` tests only.

**Fix:** Add `internal/loomrecipe/names.go` to card 15's `Edit` list, with an intent line that its doc comment names the one row table instead (a comment edit with no behaviour change), and an ImpactSummary that mentions the one production doc comment.

### [MEDIUM] Comments in two untargeted `cmd/lyx` files name the files card 11 deletes

**Location:** `_lyx/plan/11-cli-tree-walk.md`; `cmd/lyx/seamsignature_test.go:7`, `cmd/lyx/sandbox_coverage_test.go:39`

**Issue:** `seamsignature_test.go`'s file comment says "cmd/lyx/drift_test.go asserts only that every command carries a non-empty Short".
`sandbox_coverage_test.go`'s walk comment says its skip pattern "mirrors longlist_test.go's skip pattern".
Card 11 deletes both named files, yet neither referencing file is a card 11 target.
Card 4 edits `sandbox_coverage_test.go`, but it runs before card 11 and cannot name `clitree_test.go`, which does not exist yet.

**Fix:** Add both files to card 11's `Edit` list, with an intent line that their comments name `clitree_test.go`'s walk in place of the deleted files, and an ImpactSummary covering the two comment edits.

### [MEDIUM] Card 28 leaves the fate of loomcli's `commitStatusDeps` unstated while untargeted files use it

**Location:** `_lyx/plan/28-statuscommit-seam.md`

**Issue:** Card 28 says `statuscommit` "owns the deps struct (`MergeActive`, `Commit`, `Push`)" and that the two CLIs "lose their private copies".
`loomcli`'s `commitStatusDeps` is used outside the card's targets:

- `internal/loomcli/commitrecords.go`, a production file, takes it in `commitRecordsVerb(out, d commitStatusDeps)`.
- `internal/loomcli/commitrecords_test.go` builds it as a composite literal.
- The protected `wiring_commitstatus_integration_test.go` calls `newCommitStatusSeam(loomCommitStatusDeps(...))` and `newCommitStatusSeam(deps)` after reassigning `deps.MergeActive`.

The card names the integration test as "must pass untouched", but it never states that `loomcli` keeps `commitStatusDeps`, with its flat field set including `SetBoardStatus`, and keeps `newCommitStatusSeam(deps)` as the wrapper around the shared core.
An implementer who reads "lose their private copies" literally removes or reshapes the type and breaks files the card cannot edit.
An embedded `statuscommit.Deps`, for example, breaks `commitrecords_test.go`'s composite literal.

**Fix:** State in card 28's intent that `loomcli` keeps its `commitStatusDeps` type and field set and its `newCommitStatusSeam(deps)` signature as a wrapper that does the board-status write and then calls the shared core.
Name `commitrecords.go`, `commitrecords_test.go` and the protected integration test as the reason.
Only the duplicated message renderer, disposition and core closure move.

### [LOW] shedverbs' twin test comment cites the loomcli test card 27 deletes

**Location:** `_lyx/plan/27-shedverbs-duplicates.md`; `internal/shedverbs/step_test.go:24`

**Issue:** The doc comment on `shedverbs`' `TestStepEnvelope_KeySetIsExactlySixteen` says it "mirrors internal/loomcli/step_test.go's own closure test".
Card 27 deletes that `loomcli` test.
`shedverbs/step_test.go` is a card 27 target, but the intent does not say to rewrite the comment, so it would dangle.

**Fix:** Add an intent line to card 27 that comments in `shedverbs`' tests that cite a deleted `loomcli` test are rewritten to stand alone.
