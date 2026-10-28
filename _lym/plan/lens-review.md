# Lens review

Every fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates across lenses are merged into one line naming each lens.

- coverage, BLOCKING — card 1's darn row expects no history call on a record `Validate` finds clean: fixed as proposed; the repeated-id darn row now calls the history seam once and commits nothing, and a new darn row that also misses a section gets both findings, section first, with no history call.
- coverage + row-validity + guard-reach, MEDIUM — no row reaches the unpartitioned `GlyphAmbiguous` → `citation-symbol-ambiguous` branch: fixed as proposed; card 4's fixture gains `internal/real/twice_a.go` and `twice_b.go`, both declaring `func Twice()`, and a row citing `internal/real#Twice` asserts the finding names both files and gives the "cite the declaring file" way forward.
- coverage, MEDIUM — `envSeamStructs` is never checked against the real `Env`: fixed as proposed; card 6 adds a row asserting the set holds `internal/shedrecipe#Env` and `internal/landingshed#Deps` and no glyph from outside the module.
- coverage, MEDIUM — no rows for `reflect.VisibleFields`, bare `NilSeams` in package `envkit`, or `NilSeams` through a helper: fixed as proposed; card 6 adds all three rows.
- coverage + row-validity, MEDIUM — D6 covering and holding branches have no rows: fixed as proposed. Card 8 adds covering rows (the test's file under Uses, `internal/c#` and `internal/c_test#` under Uses, the card's own Edit of the test, a later Extends) and holding rows (Extends, Delete and Rename targets).
  The Delete/Rename/Move-of-the-test's-file covering forms share one code path with the Uses file row, so the Uses row reaches it.
- coverage, MEDIUM — D9 modes are exercised only for D7: fixed as proposed; cards 8 and 9 add the lens-review gate, `--lens-review`, `--overlay`, `--rework` and the negative `--require-approved` rows for their checks.
  Webster's dispatch is covered differently: each of cards 7 to 9 adds an assertion that `ValidateDispatch` over the failing row's plan reports none of its check, since dispatch runs only the index and a full webster run would test nothing more.
- coverage, MEDIUM — new refusals' way-forward text is not asserted: fixed as proposed; the rows of cards 1, 4, 6, 7, 8 and 9 now assert each finding's way-forward clause.
- coverage, LOW — D3 reads HEAD's tree rather than what git tracks, and the no-commit error has no row: fixed differently.
  The proposed fix, reading git's index, was not made, because `internal/gitrepo` has no index read (`internal/gitrepo/worktree.go` offers changed and untracked files only, and the root entries come from `SubdirsAtRevision`/`FilesInDirAtRevision`); adding one is new scope.
  Instead, card 4 records "HEAD's tree" as a stated bound, so a root entry that is staged but never committed is untested, and a worktree with no commit now has no tracked root, so no path is tested there. A row covers that case.
- coverage + row-validity + guard-reach, MEDIUM — D8 test-code branches have no row, and a test-only `Create` of a `Test…` handle is wrongly classified: fixed as proposed. Card 9 now applies the name rule to any unresolved member and adds rows for:
  - `plan:internal/d#TestNew` and `plan:internal/d_test#TestNew`, which pass;
  - the file `internal/d/new_test.go`, which passes;
  - the helper handle `plan:internal/d#newFixture`, which fails;
  - a rename of `internal/d#fixture` reaching the old-side branch alone;
  - an ambiguous `internal/d#Twice`, which counts as non-test;
  - a later Extends of `doc.go`;
  - groups naming the file only under `Prosa` or `Custom`, which are skipped.
- coverage, LOW — no row shows the darn "discussion" gate resolves without `PlanIndex` or `WorktreeRoot`: fixed as proposed; card 4 adds a darn-form row to `TestResolveGateSpec_ValidOneEntryPerVocabularyName`.
- coverage, LOW — no row asserts the loom finding order or an empty-title `### D1 — ` heading: fixed as proposed; card 1 adds both.
- coverage, LOW — no row asserts that the help texts name the new checks: disputed, ground every fix worse. The only fix is a test pinning static `Long` help prose. The Plan-Review rubric's "Test economy" flags an assertion on static prose, `PATTERN-test-speed` has a test pin behaviour rather than wording, and forced-edits found no existing help-text pin to extend. D11 is carried by the doc edits on cards 1, 4, 7, 8 and 9.
- coverage, LOW — no row for `ResolveGlyphs` returning an error wrapping `ErrQuarryUnavailable`: fixed as proposed; card 3 adds a row over a worktree root that does not exist.
- coverage + row-validity + guard-reach, BLOCKING — card 3 says fakes embedding `planindex.Index` need no change, yet `TestPlanWriteEntry_GateConfig`'s row on `noFindingsPlanIndex`, and any `FullEnv` gate on `envkit`'s `noFindingsIndex`, would panic on `ResolveGlyphs`: fixed as proposed; card 8 creates `noFindingsPlanIndex.ResolveGlyphs`, card 4 creates `noFindingsIndex.ResolveGlyphs`, and card 3's sentence now says where each is added.
- way-forward, MEDIUM — the seam reader's scan error halts the plan gates with a misplaced-seam message that names no next step: fixed as proposed; card 5 gives the seam `GuardMisplaced` error a way forward, and card 8 states the halt is an infrastructure fault fixed in the named test file by a session that may edit tests.
- way-forward, LOW — the `seam-test-unlisted` and `doc-go-unlisted` ways forward dead-end when a later card deletes, renames or moves the test, its file or the `doc.go`: fixed as proposed, by the second proposed option; both ways forward end with "else list it under Edit". The first option, widening coverage to a later Delete/Rename/Move, was not made, because D6 and D8 name only a later Edit or Extends as covering.
- way-forward, LOW — the no-commit read error in card 4 names no next step: fixed by the D3 change above; a worktree with no commit is no longer an error.
- forced-edits, MEDIUM — `pattern/PATTERN-test-speed.md:83` becomes false, but the file is under Extends: fixed as proposed; it moves to Edit, and card 5's Intent rewrites that sentence to admit `//lym:seam` lines.
- forced-edits, MEDIUM — `internal/impactset/doc.go` and `GuardScanError` keep describing three kinds and keep-only stacking: fixed as proposed; the doc moves to Edit, `internal/impactset#GuardScanError` joins card 5's Edit, and both are rewritten for the seam line and the fourth kind.
- forced-edits, LOW — the `shedrecipe.Env` field docs of `WorktreeRoot` and `PlanIndex` go stale: fixed as proposed; card 4 lists `internal/shedrecipe#Env` and updates both field docs.
- forced-edits, LOW — the file header of `internal/discussionparser/validate.go` goes stale: fixed as proposed; card 1 lists the file and its Intent rewrites the header.
- forced-edits, LOW — `internal/loomshed/doc.go`'s no-tree-sitter sentence is not widened: fixed as proposed; card 4's doc edit widens it to the citation check and names the cgo-free `quarry/glyph` grammar.
- row-validity, MEDIUM — card 4's `docs/overview.md#Modules` row has no `docs/overview.md` in its fixture: fixed as proposed; the fixture commits `docs/overview.md`.
- row-validity, MEDIUM — card 7's `TestValidatePlanCmd*` rows run without a `gate.yaml`, so `TierTags` stays nil: fixed as proposed; each new row writes a `gate.yaml` declaring `tier0` to `tier3` into its fixture's config directory, and the `--require-approved` row is paired with a no-flag row over the same plan, so it cannot pass vacuously.
- row-validity, LOW — card 8's nested-struct row repeats the direct-struct row: fixed as proposed; the row is dropped.
- row-validity, LOW — card 4's outage fake is not listed: fixed as proposed; card 4 creates `plan:internal/loomshed#unavailableIndex`.
- guard-reach, LOW — the outage row must cite a symbol to reach `ResolveGlyphs`: fixed as proposed; the row's record cites `internal/real#Thing`.
- row-validity, LOW — card 4's resolve refusals are planned in the valid-entries test: fixed as proposed; they move to `TestResolveGateSpec_Rejections`, which card 4 now lists.
- row-validity, LOW — card 7's done-card row relies on a helper that does not exist: fixed differently. The proposed choice was to duplicate or extract the setup from `TestNewPlanGate`; extracting would edit a member card 7 lists under Uses. The row instead writes its record with one `websterengine.SaveState` call, as `TestNewPlanGate`'s inline closure does, and card 7 lists `internal/websterengine#SaveState` under Uses.
- row-validity, LOW — the new top-level `TestValidatePlan_CardChecks` needs a reason not to be rows of `TestNewPlanGate`'s table: fixed as proposed; card 7 states the reason. That table's fixture writes a `language: none` plan, which the three checks skip, and its rows share no worktree quarry can resolve.
- guard-reach, MEDIUM — the `_lym`/`.lym` exclusion is never reached: fixed as proposed; card 4's fixture commits `_lym/keep.md`, and its row cites `_lym/plan/missing.md`, which only the `lymdirs` exclusion passes. `.lym` gets no row, since git never tracks it and no fixture can make it a tracked root.
- guard-reach, LOW — card 6's fixture must declare the `Env` and `Inner` structs its markers name: fixed as proposed; the fixture package declares `T`, `Env` and `Inner`.
