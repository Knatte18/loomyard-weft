```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge, for each finding fixed as proposed, whether the proposed change itself is wrong or breaks something. These are the findings I built a case for.

1. **MEDIUM — F4, `internal/fabriccli/teardownchild.go:73`.** The child now exempts itself from every pause, on the premise that the verb that handed it the job had already exempted itself.
   - Only the removal has that exemption: `fabric.go:1103` sets `exemptAll: pauserecord.SetGateExemptAll` in `remover`.
   - The cleanup sweep (`cleanup.go:278`) and the task-branch discard (`taskbranchdiscard.go:36`) also start this child. Nothing in my context shows either one exempting its own process.
   - So the new comment ("an explicit removal, discard or cleanup ... had already exempted") is unsupported for two of the three callers.
   - Under a pause, a gated `cleanup --apply` or discard would now have its remote pushes and deletes run anyway by the exempt child. That widens network access past what the verb itself had.
   - The reviewer offered a second option, keeping the child gated and documenting the wait. That option would not have this problem for those two verbs. At minimum, the exemption should be passed per job kind, or the comment and `fabriccli/doc.go` limited to the removal.

2. **MEDIUM — F6, `failedRemoteSteps`, remove kind, archive failure with `WeftTip != ""`.** The way forward is now only the tag push, with no re-run.
   - That drops the `--remote` work the job asked for: the weft branch delete and the task branch delete on origin. The notice no longer says those were skipped.
   - Case: origin already holds the weft branch, because an earlier push or the added pair was pushed. The tag push fails transiently.
   - Pushing the tag alone then leaves `<slug>-weft` and the task branch on origin forever, with a notice that reads as settled.
   - The re-run would have worked in this case: `refuseWhenNothingRemains` finds the origin branch, so it is not pair-not-found.
   - The advice should say "then run `lym fabric remove <slug> --remote` if origin still holds a branch of the pair", or the job should carry whether origin holds the branch.

3. **LOW — F5 twin, `internal/fabriccli/anchor.go:411` and `anchordeps.go:447`.** The anchor move's `EndDobbies(..., RolePush, RoleSync)` and `dobbyPassInFlight` spare lists were not given `RoleTeardown`.
   - `endOwnedProcesses` now always spares the teardown child, but the anchor move's spare lists still name only push and sync.
   - If `EndDobbies` ends non-dobby records of the owner, a prime move would end a running teardown child silently. That is the failure F5 describes.
   - I could not confirm from my context whether `EndDobbies` reaches records of other roles.

4. **LOW — F5, `warnWhenEnded`.** It calls `os.Exit(1)` from a goroutine, so `RunTeardownChild`'s deferred lock release and the exemption restore never run.
   - The OS releases the flock, so the lock itself is fine.
   - The Warn names `lym fabric remove <slug> --remote` as the way forward even for a discard child, for which F6's own analysis says a re-run is wrong.
   - The signal path has no test.

Challenged but no finding: F1 (the `failed` flag excludes `ErrLeaseRejected`, which matches the review), F2, F3, F7, F8 and F9.

Further tool calls: none.
