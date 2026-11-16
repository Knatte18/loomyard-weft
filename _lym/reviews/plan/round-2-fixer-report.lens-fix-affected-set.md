```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I checked that the fix covers every member of each set-based finding.

Unchecked findings: none. F7 has no set premise. I reran the searches for F2, F3, F4 and F5; for F1 and F6 I relied on searches already in context.

- **F1** (seam commits must write `Warp-SHA`): both commit closures are changed in card 8. The only other code-tip writer in the plan is card 12's `ReturnToPreviousTransition`, which already commits with `Warp-SHA` explicitly. Covered.
- **F2** (`reset --keep` of the parent): the plan names a parent reset only in cards 6 and 11, and both now route through `resetKeepUnderArchive`. Covered.
- **F3** (leased moves of origin's code branch): the moves are in cards 6 and 7, and both now name `updateRemoteBranch`. Card 12:59 only refers to card 6's `pushOwedCodeTip`. Covered.
- **F4 and F5** (every use of the removed `Spawn`): each one is now an edit target in card 8, or stays out of scope:
  - Production code: `readopt.go:98` is now covered, and `cancelcmd.go:484`, `commitstatus.go:119/139/184` and `wiring.go:217/339` were already covered.
  - battencli tests: `TestNewCommitStatusSeam`, `TestCommitRunRecords`, `TestCommitCancelRecord`, `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`, `TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`, `TestBattenRecordsCommitSeams`, `newFakeReceiver` and `TestStatusPushSpawns_TellTheirLocationAndGateSlug`.
  - loomcli tests: `TestNewCommitStatusSeam_OrdinaryPath`, `_BoardStatus`, `_StoppedCommitStartsNoPush` (`commitrecords_test.go:101`), `_Finish`, and now `TestPrePushCheckWiredIntoDeps`.
  - statuscommit tests: they are already in card 8's `TestNew_*` edits.
  - Out of scope: `cleanupDeps.Spawn` and `InnerRun.Spawn` are other types that card 8 does not change.
- **F6** (plan readers of `FailedPush.Run`): the only mentions were in 00-overview, card 3 and card 7, and all three now read `Runs`. Covered.

no findings

Tool calls:
1. Searched the cards other than 6 and 7 for reset and leased-move sites, because the context had searched only the touched cards.
2. Searched every `Spawn` field use in battencli, loomcli and statuscommit, because the context listed only the `.Spawn()` call sites, not the struct-literal fills.
3. Searched cards 5, 9 and 10–14 for `Warp-SHA` and commit sites, because the context had not read those cards.
4. Mapped four `Spawn` test lines to their enclosing tests, to check them against card 8's edit list. The context lacked those function names.
