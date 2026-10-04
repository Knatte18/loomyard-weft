# Card 1 — retire-pattern-residue

**Edit:**
- `internal/fabricengine/pull.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/weft_verbs.go`
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Intent:** Retire fabric's PATTERN-residue report, per the decision record's Operator additions consequence.
PATTERN moves to warp in this task, so a scan of weft commits for the PATTERN paths can only ever report nothing.

Remove `PullResult.PatternResidue`, the `PatternResidueEntry` type, `patternResidueCommits` and `parsePatternResidueRecords` from `pull.go`, together with the reconcile branch's call and its `residue` `PartialPullError` stage.
Keep `AnchorWeftSHA`, but drop the part of its doc comment that names the residue range.
Remove the `pattern_residue` key from `lyx fabric pull`'s envelope in `weft_verbs.go`, and the comment that describes flattening the entries.
Delete `TestPull_IdentifiesPatternResidue` and `TestPull_IdentifiesPatternResidueUnderSubpathAnchor`, plus any parser unit test of `parsePatternResidueRecords`.
Rewrite `doc.go`'s sentence that calls `PullResult` a PATTERN-residue report so it describes what the result still reports.

`tools/sandbox/SANDBOX-FABRIC-SUITE.md`'s step that expects the pull output to report `_lyx/PATTERN.md`-touching commits is rewritten to what the pull still reports.
Its line that expects unwire to preserve `_lyx/PATTERN.md` drops that file name, since PATTERN no longer lives under `_lyx`.

`doc.go`'s other sentence that names `_lyx/PATTERN.md` and `_lyx/pattern/` as ordinary `_lyx` content committed through `pathspec` is corrected the same way.

Grep the repository for `PatternResidue`, `pattern_residue` and `PATTERN-residue`.
Every remaining hit outside `internal/pattern` (which card 2 rewrites) and `manifest/` (which card 12 deletes) is removed in this card.
A grep for `_lyx/PATTERN` leaves hits only in card 2's files and the Go test fixtures that deliberately plant a tracked file under `_lyx`.
`pattern.PathspecFile`/`PathspecDir` keep compiling, because `junction_pattern_integration_test.go` still uses them until card 2.

**ImpactSummary:** Removes one result field, one envelope key and two private helpers in fabric; nothing outside fabricengine/fabriccli reads them.
