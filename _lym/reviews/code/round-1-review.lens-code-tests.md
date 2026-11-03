```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Pinned**
- D1 prefix refusal (empty, `dec`, `decision-`, `decision-x-`, and `design-` passing): `config_test.go` `TestLoadConfig_DesignPrefixRefused`. The test checks refusal only. The way-forward text is not asserted in the rows I saw.
- D1 render keeps the file: `decision_test.go` (Rerender keeps `decision-old.md`). The repeat-store no-write is pinned by a `readmeFile` stat in the same file.
- D2 store, repeat, absent slug, done entry, read-back: `TestStoreDecisionRecord`. Rows are `StoresBytesAndFlag`, `RunHeldEntryTakesTheStore`, `RepeatWithSameBytesWritesNothing`, `HandWrittenSameBytesStillFlags`, `RepeatWithNewBytesOverwrites`, `AbsentSlugRefused`, `DoneEntryRefused`, `RecordNotFoundOnUnflaggedEntry`, `RecordReadBackAndNotFoundWhenFileDeleted`, `RecordReadErrorReturned`.
- D3 payload rule on upsert, batch and merge: `TestDiscussedPayload` (false on a flagged entry, true on a new one, non-bool, stored value). Merge upserting a removed slug is pinned by `TestDecisionRecordFollowsEntry` `MergeUpsertingTheRemovedSlugAgainKeepsFileAndFlag`.
- D4 rename, remove, prune, merge, status clear, orphan at the target: `TestDecisionRecordFollowsEntry`. This includes the failed move reported after the save (`RenameOntoDirectoryReportsTheMoveAfterTheSave`) and the refused rename (`RenameRefusedBySeamLeavesTheFile`).
- D5: `replay_test.go` has rows for record add, change, remove, both-sides conflict, entry origin lacks, removed entry, rename and the `discussed` flag. `Claims` is covered for a root record file, a subdirectory file and an empty slug. `CommitRuns` is covered by `RecordFileNamesItsEntry`.
- D6 resolver: `entries_bouncer_test.go` was edited, but I did not read its rows. `approveseam_test.go` has the `approve_seam` map with `discussion` and the unrecognized-value row.
- D6 wiring (`TestApproveDiscussion_Real`): stores and flags the entry, done entry passes, absent entry fails, missing working copy fails.
- D7 guard test: `TestDiscussionApprovalRowsCarryApproveSeam`. It has an approval row without the key and a Code-Bouncer with the key.
- D7 `PastDiscussionApproval`: the table covers before, at and inside the segment, no status file, a row the recipe lacks, after the segment, the retired row `Merge-In`, darn, pawl, a non-Bouncer `on_done` and no `DiscussionWrite` row.
- D8: `TestDecisionVerb_CopiesRecordToBoard` has past row stores, not past row stores nothing, store failure naming the store verb, done entry without the store verb, commit error with HEAD holding both, and a status commit that landed the amendment without the entry. Way-forward text is tested at past and not-past rows.
- D9 `TestDecisionStoreVerb`: unnamed shell, darn run, inside Discussion-Review, done entry, committed record missing, status read error, store failure, named task session, empty parent name, parent error, arbiter, parent orch.
- D10 `stepGetRecord`: the flag shown and omitted, verbatim record, undiscussed entry, absent entry, flagged entry with the file deleted, `--record` beside `--body`.

**Gaps**
- F1, LOW, scope. Location: `internal/loomcli/decision_test.go`. No `TestDecisionVerb_CopiesRecordToBoard` row I saw covers a darn run or a beam project record not being copied, although D8's acceptance names both. `RowNotPast` covers only tapestry rows. `PastDiscussionApproval` does pin darn and pawl as false, so only the wiring in `decisionVerb` is unpinned. The hidden `TestDecisionVerb_CopiesRecordToBoard` row list may include one; that was not verifiable within the call budget.
- F2, LOW, scope. Location: `internal/shedadapters` Bouncer tests. No new test pins that `ApproveDiscussion` runs on the accepted-round, skipped and zero-round paths, or runs after the carry-over and before the commit seam. The diff reuses the existing `Approve` seam, so any such coverage is the plan-seam tests, and I did not confirm they exercise a non-plan seam. This is likely covered and is raised only as unverified.
- F3, LOW, scope. Location: `internal/boardengine`. A failed removal of a removed entry's record file is only logged (`followDecisionRecords`) and no test reaches it. The failure is hard to provoke.
- F4, LOW, scope. Location: `internal/boardengine/config_test.go`. `TestLoadConfig_DesignPrefixRefused` asserts the refusal; I did not see it assert the way-forward text (`set design_prefix`). `PATTERN-refusal-way-forward` for D1 is satisfied only if the message is asserted.

Tool calls:
1. Listed the added test function names and table rows from the diff. My context held no test files.
2. Grepped `decision_test.go`, the boardengine decision tests and `verbs_test.go` for rows on store, copy, darn and render. This confirmed the unnamed-shell, darn and Rerender rows my first listing missed.
