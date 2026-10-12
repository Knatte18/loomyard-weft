# Lens review

Every fork returned a report.

- criteria-coverage, MEDIUM, no row reaches a covered line inside a later region: fixed, Card 1 adds the `coveredTargetCard` row followed by a card editing `callers#UseTarget` to `TestGlyphChain_CallerCoverage`.
- criteria-coverage, MEDIUM, spec row 32 and the `doc.go` bullet listed without the new wording: fixed, Card 1's Intent states both rewrites.
- criteria-coverage, LOW, no assertion that dispatch and Webster spawn no load: rejected, `ValidateDispatch` and `LaterDeleteReferences` take no loader and pass nil, so no load is reachable, and the typed rows' scan-only dispatch verdict is the observable proof; Card 1 now says so.
- criteria-coverage, LOW, the `ErrOverlayFileMissing` way forward points at a flag-less run that fails to parse: fixed, Card 5 names writing the missing card file.
- criteria-coverage, LOW, Decision 6's "each slotted" dropped: fixed, Card 5 states the overlay validates through `c.env.PlanIndex` and Card 6's bound says each run is slotted.
- forced-edits, MEDIUM, `coverageFiles` lacks the typed rows' new files: fixed, Card 1 adds them.
- forced-edits and row-reachability, MEDIUM, the informational detail lacks the later card's ID that `TestLaterDeleteReferences`' loop requires: fixed, Card 1's informational detail names each later card's ID.
- forced-edits, LOW, spec row 32 and `doc.go` wording: merged with the criteria-coverage finding above.
- forced-edits, LOW, `CanonicalizeHandles`' doc comment still says "planDir's bytes": fixed in Card 4.
- forced-edits, LOW, `docs/overview.md` lists `validate-plan`'s modes without `--overlay`: fixed, Card 5 lists and edits it.
- forced-edits, LOW, the `ErrOverlayFileMissing` way forward: merged with the criteria-coverage finding above.
- forced-edits, LOW, the removed-card row reaches the sentinel only when that card is overlaid: fixed, Card 5's row overlays that card.
- row-reachability, MEDIUM, the informational expectations carry no line number: fixed, Card 1's expectation shape is `<file> at line <n>` for informational findings, and the rows name their lines.
- row-reachability, MEDIUM, the ignored-directory row cannot fail: fixed, the region is `other/testdata/x.go`, which imports `x/sub` and calls `sub.Foo()`, and the row also runs a `sub#` Delete, which covers `laterImportFindings`' ignored-directory skip; the absent-file row runs both Delete shapes too.
- row-reachability, LOW, the member-delete `.md` rows cannot fail: fixed, both dropped; only the `sub#` and `other#` import-path rows remain, the one path a `.md` region could reach.
- row-reachability, LOW, typed rows leave caller-uncovered `want` unstated: fixed, each new typed row names its `want`.
- row-reachability, LOW, `TestEditedDoneCards` needs an on-disk plan: fixed, Card 4 names `plankit.Write`, `ParsePlan` and the recorded SHA-256.
- row-reachability, LOW, the disk-overview parse error under `--overlay` has no wording or row: fixed, Card 5 routes it to the existing parse refusal and adds a row with no plan on disk.
- test-economy, LOW, Card 5 repeats Card 3's name rows: fixed, Card 5 keeps one `ErrOverlayName` row and the missing-file row.
- test-economy, LOW, Card 3's `Plan.ReadFile` disk row is redundant: fixed, dropped.
- test-economy, LOW, the plan-gate `other#` row duplicates `TestLaterDeleteReferences`: rejected, it is the only row that pins `planGatePass` calling `laterImportFindings` now that `resolvePass` no longer runs the check.
- test-economy, LOW, Card 5's handle row duplicates Card 4's subtest: rejected, the CLI row pins that the verb hands the overlaid plan through `loomshed.ValidatePlan` to its envelope, a surface the planglyph subtest does not reach.
