```yaml
lens: fix-proposal
outcome: reported
```

Task: the fix-proposal lens on F1, the restore of the seven archive-tag tests.

The proposal is mostly sound. Five of the seven pin a path no kept test reaches:
- the reuse path with exactly one tag on origin
- the force path
- the rollback negative
- the drive-report content
- no origin tag after a refused Remove

I found one finding.

**[MEDIUM]** `TestRemove_RemoteFalseStillPushesArchiveTag` and `TestRemove_ArchivesWeftTipBeforeTeardown` (`internal/fabricengine/remove_archive_integration_test.go`)
- **Summary:** The two restored tests are the same scenario, so one is redundant. The review's own fix text allowed folding in this case, and the fixer restored both verbatim.
- **Evidence:**
  - Both build the same hub: `CopyHub` and `AddPairWith` with default options, then `CommitFile` of `_lym/record.txt` with `"run record\n"`.
  - Both call `topology.Remove(l, slug, false, false)`, so `remote=false` is the default call and not a distinct path.
  - Both assert that the origin tag `archive/<slug>/<tip[:12]>` points at `tip`.
  - The happy-path test also asserts `res.ArchiveTag` and `ArchiveSkippedReason`.
  - The remote=false test adds only `RemoteBranchDeleted == false`.
  - The `testing` skill counts a redundant test as a defect, and a case that differs only by one assertion is a row or an assertion in the existing test.
- **Fold:** move `if res.RemoteBranchDeleted { ... }` into `TestRemove_ArchivesWeftTipBeforeTeardown`, delete `TestRemove_RemoteFalseStillPushesArchiveTag`, and extend the keep reason to name the unchanged remote branch.

No other finding. The `TestRemove_ArchivesWeftTipBeforeTeardown` assertions on `res.ArchiveTag` and the origin tag target overlap `BranchIsPushedFirst` in `TestRemove_ArchiveOrder`. The restored test still adds `ArchiveSkippedReason == ""`, and it runs in parallel where `ArchiveOrder` does not, so I don't call it redundant.

Tool calls made: none. My context already held the diff, the final file contents and the review.
