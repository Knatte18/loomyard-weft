```yaml
lens: code-tests
outcome: reported
```

Per behaviour, the test that pins it:

1. **A beam with no owner of its own is covered by an unfinished `root` adopt, watcher side.**
   - Pinned by `TestUnderAdoptWatch_CoversABeamWithNoOwnerOfItsOwnWhileARootAdoptIsUnfinished` (`internal/orchcli/underadopt_integration_test.go`).
   - It would fail if the beam read uncovered: the `Covers(b)` check under a running root record fails.
   - The setup is not vacuous. The board JSON has no `owner` key, `beamOwnerElsewhere` is asserted to return the foreign prime owner, and `Covers` is asserted false before the record exists.
2. **The same coverage, dobby side.**
   - Pinned by `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` (`internal/beamcli/dobbydeps_integration_test.go`).
   - It asserts `deps.Owner()` is the foreign owner and `UnderAdopt()` flips false→true when the record is written.
   - This goes through the real `hubDobbyDeps` seam. A broken coverage read would fail it.
3. **A code owner archive tag is judged against the code branch only.**
   - Pinned by the new decoy row `ACodeTagIsListedThoughTheRecordsBranchNameOnTheCodeOriginHoldsIt` in `TestReadOwnerArchives` (`internal/fabricengine/ownerarchive_integration_test.go:177`).
   - If the read used the records branch name, the decoy would contain the tip and the tag would drop out of the listing, so the row fails.
4. **A records owner archive tag is judged against the records branch only.**
   - Pinned by the new decoy row `ARecordsTagIsListedThoughTheCodeBranchNameOnTheRecordsOriginHoldsIt`.
   - Its failure mode is the mirror of the code row.
   - Both decoy rows add coverage beyond the existing rows. The existing rows use `pushTip` against the correct branch only and never put the tip on the other side's branch name.
5. **A teardown leaves an owner archive tag alone, still archives under `archive/`, and removes the worktree.**
   - Pinned by `TestRemove_LeavesAnOwnerArchiveTagOutOfTheTeardownArchive` (`internal/fabricengine/remove_archive_integration_test.go`).
   - `res.ArchiveTag` must carry the `archive/` name.
   - The origin must hold exactly that one `archive/` tag, and the owner tag must stay at the tip. `archiveTagsAt` lists `archive/*`, which does not match `archive-owner/…`.
   - The worktree path must be gone.
   - The setup cannot pass vacuously. If `ArchiveOwnerTips` did not push the owner tag, `tagTargetAt` would return empty and the test would fail.

Findings:

- **LOW, `scope`.**
  - Location: `internal/fabricengine/ownerarchive_integration_test.go:177-181` (the diff's only records-tag-versus-branch pin).
  - Summary: the folded note `records-tag-vs-branch-untested` asks for a test row for each way the two differ "under `lym adopt --here`".
  - The diff's rows pin the `--archives` read, `ReadOwnerArchives`, only. No test in the diff drives `lym adopt --here` with a records tag against the records branch.
  - The committed `internal/adoptcli` code I saw covers the `--archives` read only: `deps.go:174`, `doc.go:133-135`. It has no `--here` handling of the records tag.
  - The change description does not mention this reading of the note.
- **No other behaviour lacks a test.**
  - The summary says the hold's skipped steps, the tip decision rows and the end-to-end pairs are already pinned.
  - The existing tests I saw match that: `TestRun_ForeignOwner` and `TestRun_AnAdoptHoldSchedulesNothingUntilItLifts`, `TestArchiveOwnerTips` (absent, ahead and diverged rows), and `TestArchiveOwnerTips_CoversTheWorktreesOwnPairAndTheRunsUnderIt`.
  - I did not check `TestArchiveOwnerTips` for an "equal" row. The `TipsOriginHoldsArchiveNothing` name and `ATipOriginIsAheadOfArchivesNothing` suggest equal and ahead rows exist.
- **No test-economy duplicates in the diff.** Each new test or row asserts something no existing test covers.

Tool calls:
1. `grep` over `internal/adoptcli/*.go` for records-branch and owner-archive handling. My context had only `archives_test.go`, not the `--here` production code.
