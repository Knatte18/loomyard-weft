```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, covering each function, branch and error path the diff adds, run against one concrete input. I found no defects.

**List (case run, verdict)**
- `TestUnderAdoptWatch_CoversABeamWithNoOwnerOfItsOwnWhileARootAdoptIsUnfinished` (`internal/orchcli/underadopt_integration_test.go`)
  - `beamOwnerElsewhere` for beam `b` with an empty owner: `owners.Effective` falls back to the parent, the prime, so it returns `other-machine` and elsewhere is true. Correct.
  - `Covers("b")` with no record: `Read` finds none, so false. Correct.
  - `Covers("b")` under a running `root` record: `ownerSubject` sees `b` held, not listed, with no own owner. It recurses to the parent, the prime with the empty slug, which gives `root`, and `record.Covers("root")` is true. Correct.
  - Would it fail if `b` were read uncovered? Yes, the final `Covers` assertion would fail. Not vacuous.
- `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` (`internal/beamcli/dobbydeps_integration_test.go`)
  - The board has `prime_owner` set to a foreign machine and `b` with no owner, so `deps.Owner()` is the prime's owner. Correct.
  - `UnderAdopt()` is false with no record and true after the `root` record is written, through the same `Watch.Covers` path. Correct and non-vacuous.
- `TestReadOwnerArchives`, row "ACodeTagIsListedThoughTheRecordsBranchNameOnTheCodeOriginHoldsIt" (`internal/fabricengine/ownerarchive_integration_test.go`)
  - The tip is force-pushed to `RecordsBranchName(codeBranch)` on the code origin.
  - Production judges the code side against the code branch only, whose origin tip lacks the tag's commit, so the tag is listed. Correct.
  - Were the wrong branch name read, the tip would be contained and the tag dropped, so the row would fail. Non-vacuous.
- `TestReadOwnerArchives`, row "ARecordsTagIsListedThoughTheCodeBranchNameOnTheRecordsOriginHoldsIt"
  - The tip is force-pushed to `codeBranch` on the records origin.
  - The records side is judged against `RecordsBranchName(codeBranch)`, whose origin tip was pushed earlier and lacks the new commit, so the tag is listed. Correct and non-vacuous.
  - `--force` is needed only if the decoy branch already exists. It is harmless otherwise.
  - Local tag presence and the absence of teardown tags are asserted as before. Correct.
- `TestRemove_LeavesAnOwnerArchiveTagOutOfTheTeardownArchive` (`internal/fabricengine/remove_archive_integration_test.go`)
  - The records tip is unpushed and `ArchiveOwnerTips` tags it. The tip is asserted to be in the returned tags, so the setup cannot pass without an owner tag.
  - `Remove(..., false, false)` then archives under `archive/<slug>/<sha12>`.
  - `archiveTagsAt` lists `archive/*`, which does not match `archive-owner/...`, so "exactly [wantTag]" is a valid exclusion check.
  - `tagTargetAt` on the owner tag requires it to exist on origin at `tip`, so it fails if the owner tag was removed or never pushed. Non-vacuous.
  - The worktree `Stat` expecting `fs.ErrNotExist` is correct.

Two points I could not confirm without a run:
- That `hubgeom.ParentOf` returns the prime for a pair made by `AddPairWith` with `Beam: true` (inferred from the `Owner()` and `beamOwnerElsewhere` assertions).
- That `adoptrecord.Record{State: StateRunning}` counts as `Unfinished()`.

Neither is a finding. Both are the verify gate's to report if wrong.

Tool calls: none; my context already held every file I needed.
