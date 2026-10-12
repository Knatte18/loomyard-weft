# Card 4 — overlay-through-checks

**Create:**
- `internal/websterengine#cardHashesFrom`

**Edit:**
- `internal/planglyph#CanonicalizeHandles` -> `func CanonicalizeHandles(plan *planparser.Plan, worktreeRoot string, results []quarry.ResolveResult) ([]Finding, bool, error)`
- `internal/planglyph#resolvePass`
- `internal/planglyph#TestCanonicalizeHandles_ReportsWhetherItRewrote`
- `internal/planglyph#TestCanonicalizeHandles_Rewrites`
- `internal/planglyph#TestCanonicalizeHandles_OneDraftTwoCanonicalsRewritesNothing`
- `internal/planglyph#TestCanonicalizeHandles_RenameOldUnresolved`
- `internal/planglyph#TestCanonicalizeHandles_RenameOldSelfGlyphNamesTheShapeMistake`
- `internal/planglyph#TestCanonicalizeHandles_OneFailingDeclarationLeavesOthersRewritten`
- `internal/planglyph#TestCanonicalizeHandles_CanonicalCollisionRewritesNeither`
- `internal/planglyph#TestCanonicalizeHandles_LanguageNoneNoOp`
- `internal/websterengine#EditedDoneCards` -> `func EditedDoneCards(plan *planparser.Plan, st *State) ([]string, error)`
- `internal/websterengine#batchCardHashes`
- `internal/websterengine#TestEditedDoneCards`
- `internal/loomshed#ValidatePlan`

**Intent:**
Route every plan-file read and write of the plan gate's check path through the plan's own source, so an overlaid plan is checked exactly as it would be on disk and writes nothing (Decision 5, the consumers' half).

`CanonicalizeHandles` applies its substitutions with `plan.RewriteRefs(subs)` instead of `planparser.RewriteRefs(planDir, subs)`, so an overlaid plan is rewritten in memory and a disk plan on disk as today; it drops its planDir parameter, since the plan's own source now decides where the rewrite lands, and every caller stops passing it; its `rewrote` report is unchanged, and its doc comment says it rewrites the plan's card files through the plan's source and that `rewrote` reports whether that source was handed a substitution, no longer "planDir's bytes".
`resolvePass` re-reads the plan after a rewrite with `plan.Reparse()` instead of `planparser.ParsePlan(planDir)`, so the checks after canonicalization see the overlay's canonical handles; its error wording stays.
`cardHashesFrom` is `batchCardHashes`' body over a reader, the card file name derived as today; `batchCardHashes` calls it with a disk reader joined to `planDir`, so its other callers are untouched.
`EditedDoneCards` drops its `planDir` parameter and hashes through `cardHashesFrom` with `plan.ReadFile`, so an overlaid done card counts as edited exactly as the same edit on disk would; `ValidatePlan`, its one production caller, passes the plan alone.

Tests:
`TestCanonicalizeHandles_ReportsWhetherItRewrote` gains a subtest: a plan whose card on disk carries no handle, overlaid through `planparser.NewOverlay` with a card declaring `plan:sub#Draft` -> `func Actual() {}`, reports a rewrite, leaves the card file on disk byte-identical, and `Reparse` of the overlaid plan holds `plan:sub#Actual`.
`TestEditedDoneCards` passes the plan alone and gains a case: a one-card plan written to disk with `plankit.Write` and parsed with `planparser.ParsePlan`, and a done batch recorded with the SHA-256 of that card file's bytes, report nothing for the disk plan and report the card for the plan `planparser.NewOverlay` builds with different text for that card.

**ImpactSummary:** `EditedDoneCards` has one production caller, `loomshed.ValidatePlan`, and one test; the other `batchCardHashes` callers in begin-batch, recover-batch, rebaseline and the fingerprint keep reading disk; `CanonicalizeHandles`' callers, `resolvePass` and its own tests, all in `planglyph`, drop the planDir argument and see no behavior change for a disk plan.

**Commit:** 4: plan gate reads and canonicalizes through the plan's own source
