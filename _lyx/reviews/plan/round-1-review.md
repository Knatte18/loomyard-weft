---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/01-shared-reference-delete-order.md:24
    summary: Card 1 lists internal/planglyph#deleteOrderFixture under Edit, but its Intent changes nothing in that fixture map; every new TestLaterDeleteReferences row puts its files in the row's extraFiles
    fix: In _lyx/plan/01-shared-reference-delete-order.md, delete the Edit bullet `internal/planglyph#deleteOrderFixture`. Card 1 still lists `internal/planglyph#TestLaterDeleteReferences` from the same test file, so no file path is needed in its place.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/04-overlay-through-checks.md:18
    summary: Card 4 routes CanonicalizeHandles' only use of its planDir parameter through plan.RewriteRefs but keeps the signature unchanged, leaving planDir a dead parameter that callers still pass and that no longer decides where the rewrite lands
    fix: In _lyx/plan/04-overlay-through-checks.md, replace the Edit bullet `internal/planglyph#CanonicalizeHandles` with `internal/planglyph#CanonicalizeHandles` -> `func CanonicalizeHandles(plan *planparser.Plan, worktreeRoot string, results []quarry.ResolveResult) ([]Finding, bool, error)`; add Edit bullets for the test callers `internal/planglyph#TestCanonicalizeHandles_Rewrites`, `internal/planglyph#TestCanonicalizeHandles_OneDraftTwoCanonicalsRewritesNothing`, `internal/planglyph#TestCanonicalizeHandles_RenameOldUnresolved`, `internal/planglyph#TestCanonicalizeHandles_RenameOldSelfGlyphNamesTheShapeMistake`, `internal/planglyph#TestCanonicalizeHandles_OneFailingDeclarationLeavesOthersRewritten`, `internal/planglyph#TestCanonicalizeHandles_CanonicalCollisionRewritesNeither` and `internal/planglyph#TestCanonicalizeHandles_LanguageNoneNoOp` (resolvePass and TestCanonicalizeHandles_ReportsWhetherItRewrote are already listed); in the Intent, replace "its signature and its `rewrote` report are unchanged" with "it drops its planDir parameter, since the plan's own source now decides where the rewrite lands, and every caller stops passing it; its `rewrote` report is unchanged".
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/01-shared-reference-delete-order.md:55
    summary: Card 1 says scan-only LaterDeleteReferences runs never walking the tree, but the coverageSubjects call it prescribes runs planparser.NestedModules, whose goModDirsOnDisk walks every directory under the worktree looking for go.mod
    fix: In _lyx/plan/01-shared-reference-delete-order.md line 55, replace "never walking the tree" with "never walking the tree for candidate files (coverageSubjects' go.mod directory walk for the import path still runs once)".
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/01-shared-reference-delete-order.md:9
    summary: Card 1 declares countingTypesLoader and its load method as plain internal/planglyph handles without saying they are test-only, though only the integration-tagged TestGlyphChain_CallerCoverageTyped uses them; placed in a production file they would ship as dead code
    fix: In _lyx/plan/01-shared-reference-delete-order.md, in the Intent's TestGlyphChain_CallerCoverageTyped bullet, after "which counts its calls and forwards them", add "and which, with its load method, lives in glyphchain_integration_test.go beside the test".
---

The plan carries every Decision and Constraint in the decision record.
Card order, verify coverage and order, re-sign arrows, attack-surface bounds and refusal reachability all hold.
The four findings are local wording and target-list fixes.

### [LOW] Card 1 lists an Edit target its Intent never changes

**Location:** `_lyx/plan/01-shared-reference-delete-order.md:24`

**Issue:** `deleteOrderFixture` is a package-level map in `internal/planglyph/deleteorder_test.go:16`.
Card 1's Intent says "The extra files go in the rows' `extraFiles`".
The rows that reuse fixture files (`sub/a.go` for the own-declaration row, `sub/d.go` for the method row) read them unchanged.
So the bullet names a member the card does not change, and the implementer has to guess what change was meant.

### [LOW] Card 4 leaves CanonicalizeHandles' planDir parameter dead

**Location:** `_lyx/plan/04-overlay-through-checks.md:18`

**Issue:** `CanonicalizeHandles(plan, planDir, worktreeRoot, results)` uses `planDir` only at `internal/planglyph/handle.go:345`, `planparser.RewriteRefs(planDir, subs)`.
Card 4 replaces that call with `plan.RewriteRefs(subs)` and declares the signature unchanged.
After card 4, the parameter is still passed by `resolvePass` and eight test functions but decides nothing.
A caller passing a directory other than `plan.Dir` would have it ignored without any sign.
Decision 5 licenses routing this rewrite through the plan's source, so dropping the leftover parameter is in scope.
The fix's caller list comes from `grep CanonicalizeHandles(` over `internal/planglyph`.

### [NIT] Card 1's "never walking the tree" claim is inaccurate

**Location:** `_lyx/plan/01-shared-reference-delete-order.md:55`

**Issue:** `coverageSubjects` (`internal/planglyph/callercoverage.go:195`) calls `planparser.NestedModules(plan, 0, worktreeRoot)` on its first subject.
That calls `goModDirsOnDisk` (`internal/planparser/nestedmodules.go:100`), a `filepath.WalkDir` over the whole worktree.
The claim holds for the candidate-file walk `candidateFiles` does, but not for the subject build the same sentence prescribes.

### [NIT] countingTypesLoader's home is unstated

**Location:** `_lyx/plan/01-shared-reference-delete-order.md:9`

**Issue:** Card 1's Create handles carry only the package unit.
The Intent describes `countingTypesLoader` purely as a wrapper inside `TestGlyphChain_CallerCoverageTyped`, which sits in the integration-tagged `glyphchain_integration_test.go`.
Nothing says the type belongs in a test file.

## Rejected

- Re-sign arrows (focus 1): settled per glyph against the code.
  `planGatePass`, `callerCoverageFindings`, `loadTypedReferences` (a nil `loader` is a body change), `candidateFiles`, `resolvePass`, `ValidateDispatch`, `ValidateFormatAfter`, `validateFormatAfter`, `LaterDeleteReferences`, `laterDeleteReferenceReasons`, `batchCardHashes`, `checkCardNumbering`, `RewriteRefs` and `loomshed#ValidatePlan` keep their signatures.
  `Index.LaterDeleteReferences` is an interface method and `Plan` gains only a struct field.
  `Plan.RewriteRefs` is a Create.
  The re-signed `rewriteCardFile`, `cardHeadingNumber`, `EditedDoneCards` and `deleteReferencedBatches` all carry the arrow, and each one's callers are on the same card.
- Card 1 granularity (focus 2): every Create is unexported and serves the one delete-order and caller-coverage surface in `planglyph`.
  `dispatchFindings` is `ValidateDispatch`'s extracted body, and `countingTypesLoader` serves one test.
  Splitting the card would leave the shared computation half-wired between commits.
- Test economy (focus 3): cards 1 and 2 add rows to existing tests.
  The new covered-line row differs from the existing "a later card editing a member partitioned by build constraints" row, whose later-region line no admissible target covers.
  `TestOverlay` and `TestValidatePlanCmd_Overlay` pin new surfaces no existing test reaches.
  Card 4's overlay subtest is that card's own coverage, needed before card 5 exists.
- Attack surface (focus 4): card 1 states the informational downgrade and its bound.
  Card 2 states the Webster bound, card 3 the closed name set, and card 5 what `--overlay` lets through (no writes, no tree change, no file add or remove).
- Reachability (focus 5): every prescribed refusal names an input that passes the upstream checks and a test that reaches it.
  That covers the five overlay refusal classes (card 5 rows and card 3 `TestOverlay`), `ErrOverlayFileMissing` (the overlaid removed card, which `NewOverlay`'s overview-only parse does not reject first), and Webster's blocking-only filter (the `internal/foo#Thing.Gone` rows, whose scan sees `t.Gone()` as name-only).
- Fidelity (focus 6): Decisions 1–7 and every Constraint map onto cards 1–6.
  `docs/overview.md` (card 5) already describes `validate-plan`'s modes at lines 431–432, so updating it keeps the licensed flag's docs true; it adds no scope.
- Verify (focus 7): `./...` under the untagged and integration tiers covers every targeted package, `cmd/lyx` included.
  The repository has one module, and the section follows the template's order.
- Card order (focus 8): cards 4 and 5 consume card 3's API, card 5 consumes card 4's re-signed `EditedDoneCards`, card 6 consumes card 5's flag, and card 2 consumes card 1's severities.
  No card depends on a later card.
