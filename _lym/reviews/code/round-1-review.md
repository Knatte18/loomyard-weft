---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: handler
    location: internal/fabricengine/remove_archive_integration_test.go
    summary: Seven dropped tests assert archive tags on origin, a category the record says to keep with a testtiming:keep reason
    fix: Restore these tests from f2ac2bf7 (git show f2ac2bf7:<file>) and mark each with a testtiming:keep line naming the archive tag it pins, re-adding any helper the restore needs (archiveTagsAt, showAtTag). In remove_archive_integration_test.go, TestRemove_ArchivesWeftTipBeforeTeardown, TestRemove_ReusesSameTipArchiveTag (exactly one tag on origin), TestRemove_ForceStillArchives, TestRemove_RemoteFalseStillPushesArchiveTag and TestAddRollback_LeavesNoArchiveTag (no tag locally or on origin). In remove_siblingdirty_integration_test.go, TestRemove_UntrackedDriveReportIsCommittedAndArchived. In remove_records_integration_test.go, TestRemove_OutOfPathspecSiblingDirtRefusesBeforeAnyMutation (no origin tag after a refused Remove). Where a kept test already pins the same tag assertion, fold the restored case into it as a row instead and say which one covers it in the commit message.
---

### [BLOCKING] Dropped tests assert archive tags the record says to keep

**Location:** internal/fabricengine/remove_archive_integration_test.go, remove_siblingdirty_integration_test.go, remove_records_integration_test.go
**Issue:** The decision record's keep rule lists "archive tags and their order" as something block coverage cannot see, and says such a test carries `//testtiming:keep <reason>`.
All of these tests are in `greedy.tsv`, so the set licenses dropping them, but the record asks the writer to read each one first and keep it when it asserts a tag.
The removed lines show `wantTag := "archive/" + slug + "/" + tip[:12]` compared with `res.ArchiveTag`, `archiveTagsAt(t, h.RecordsBare)` expected to equal exactly `[wantTag]` in the reuse test, and `archiveTagsAt` expected empty after a rolled-back Add or a refused Remove.
Only a few archive tests were kept; the summary's keep list names none of these seven.
I did not run the kept tests to confirm none of them also pins the reuse path, the force and remote=false paths, the rollback negative or the drive-report content, so the fix allows folding into a kept test where that is true.

## Rejected

- Fork lens reports (code-correctness, code-affected-set, code-tests) returned no findings, so nothing was rejected.

## Checks that came out clean

- All 198 dropped tests are in `greedy.tsv`, none is duplicated, and none overlaps the first cut's drops.
- The 4 unmarked greedy tests still present are the ones the summary names as folded by the first cut.
- The 54 added keep lines in `internal/fabricengine` match the summary's kept count, and each reason names what it pins.
- The diff touches only test files; `go vet ./internal/fabricengine` is clean, so no import or helper is left unused or missing.
- `cmd/lym/cwdmutation_test.go`: the emptied allowlist matches the coalesce test having no `Chdir` token, and no entry for a cwd-changing test was dropped.
