```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM — `15-snapshot-reader.md`, "Parts of tests deleted", `TestMergeState_Record` bullet: the holding test is misnamed.**
- The card says "`TestMergeSiblings_Dispositions` [holds] the record's absence".
- The record-absence assertion is at `internal/fabricengine/mergesiblings_integration_test.go:268-270`, and the test enclosing it is `TestMergeSiblings_CommitRefusesForeignMergeState`.
- The bullet should name that test.

**MEDIUM — `15-snapshot-reader.md`, "Whole tests and helpers deleted", `TestRemove_ForceStillArchives` bullet: the holder is weaker than the card claims.**
- The card says "`remove_records_integration_test.go`'s forced remove asserts the archive tag". That forced remove is in `TestRemove_OutOfPathspecSiblingDirtRefusesBeforeAnyMutation`, and at `remove_records_integration_test.go:133-142` it only checks that `ArchiveTag` is non-empty and what file content sits at the tag.
- The deleted test pins more: the exact name `archive/&lt;slug&gt;/&lt;tip[:12]&gt;`, and origin's tag pointing at the records tip (`remove_archive_integration_test.go:256-264`, keep reason "the archive tag pushed to origin at the weft tip").
- So the deletion loses the tag-name and tag-target assertion under a force remove, and the card's justification overstates what the holder covers.

Claims I checked that hold:
- **`TestCommit_TwoSided_WarpFirstOrdering`:** `TestCommitWithTrailers/TrailersEndTheWeftMessageOnly` asserts both sides committed and that the weft message ends with the `Warp-SHA: &lt;WarpSHA&gt;` trailer (`commit_integration_test.go:325-332`).
- **`TestDropHandOverMark`:** the `WritesTheEntry` subtest writes a mark and asserts it is dropped (`takeoverrun_integration_test.go:90,106-108`). The `APauseHolds` subtest writes no mark and wants a nil error.
- **`TestNew_HappyPath`:** `Open` returns `newPaired(...)` (`open.go`), `TestOpen` has a `happy_path` row (`fabric_test.go:43`), and `newPaired` always returns a non-nil `&amp;Fabric` on success (`fabric.go:51-65`).
- **`TestCheck_HasNoFourthMemberForForce`:** it checks `len` of a literal it builds itself (`refusalof_test.go:121-125`).
- **`TestGate_*`:** the cited row and subtest names exist in `destroy_test.go`: `AbsoluteOutsideContainer`, `WithinContainerPassesContainment`, `PathEmptyNAReasonWithAbsentTarget`, the `WithAbsentTarget` rows and `RemovePath_AbsentTargetRecordsNothing`.
- **Card 15's reason that `gitrepo`'s open refuses a non-repository path:** it matches the doc comment at `internal/gitrepo/gogit.go:58-60`.

Tool calls:
1. `commit_integration_test.go:312-340`, `fabric_test.go`, `fabric.go`, `mergesiblings_integration_test.go:260-272`, the `takeoverrun` subtest list, `refusalof_test.go:113-126` and the `destroy_test.go` row names. My context held only the search agent's citations, not the code.
2. `takeoverrun_integration_test.go:86-131`, the name of the test enclosing `mergesiblings:268`, `commit_integration_test.go:29-77`, `newPaired` and `Open`. Needed to verify the holder claims.
3. `takeoverrun_integration_test.go:338-368`, `remove_records_integration_test.go:128-145` and `remove_archive_integration_test.go:240-266`. Needed to verify the hand-over-mark and forced-remove holders.
