# Card 1 — retire-pattern-residue

**Edit:**
- `internal/fabricengine/pull.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/weft_verbs.go`

**Intent:** Retire fabric's PATTERN-residue report, per the decision record's Operator additions consequence.
PATTERN moves to warp in this task, so a scan of weft commits for the PATTERN paths can only ever report nothing.

Remove `PullResult.PatternResidue`, the `PatternResidueEntry` type, `patternResidueCommits` and `parsePatternResidueRecords` from `pull.go`, together with the reconcile branch's call and its `residue` `PartialPullError` stage.
Keep `AnchorWeftSHA`, but drop the part of its doc comment that names the residue range.
Remove the `pattern_residue` key from `lyx fabric pull`'s envelope in `weft_verbs.go`, and the comment that describes flattening the entries.
Delete `TestPull_IdentifiesPatternResidue` and `TestPull_IdentifiesPatternResidueUnderSubpathAnchor`, plus any parser unit test of `parsePatternResidueRecords`.
Rewrite `doc.go`'s sentence that calls `PullResult` a PATTERN-residue report so it describes what the result still reports.

Grep the repository for `PatternResidue`, `pattern_residue` and `PATTERN-residue`.
Every remaining hit outside `internal/pattern` (which card 2 rewrites) and `manifest/` (which card 12 deletes) is removed in this card.
`pattern.PathspecFile`/`PathspecDir` keep compiling, because `junction_pattern_integration_test.go` still uses them until card 2.

**ImpactSummary:** Removes one result field, one envelope key and two private helpers in fabric; nothing outside fabricengine/fabriccli reads them.
