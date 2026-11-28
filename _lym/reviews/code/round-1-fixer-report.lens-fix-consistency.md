```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F3 change set.

1. **MEDIUM**: `_lym/landing/summary.md`, `## Merge danger`, "The detached push already sent that commit there, so the synchronous push puts nothing new on origin."
   - The code does not hold this. `Fabric.commit` only spawns a fire-and-forget child (`_ = spawnDetachedPushFn(pushRequest)`, `internal/fabricengine/commit.go:169-172`). The child may not have run yet, may fail to start or may fail to push, so the synchronous push can be what first puts the commit on origin.
   - It contradicts the same document's opening: the commit "was then left to the detached pusher's quiet retry: a push that failed wrote no failed-push record".
   - The `two-way` door call rests on this sentence. A true form is that a detached push of the same commit was already spawned before this change. The door needs to stand on that, or change.

2. **LOW**: `internal/webstercli/beginbatch.go:25`, the comment on `fabricSyncWayForward`.
   - It says the constant is "the trailing clause of every refusal raised because the fabric sync failed".
   - The new hand-over branch in `reset.go` raises such a refusal without it. The twin comment was left unchanged.

3. **LOW**: `internal/webstercli/reset_test.go:1-3`, the file header.
   - It says "each refusal checks its way forward and that nothing moved".
   - Neither subtest of the new `TestResetCmd_SyncFailureAndTheRecordsPush` checks that nothing moved, and neither can, since these refusals follow a completed reset. One of them asserts that the sync way forward is absent.

The help text, the `doc.go` Reset section, `pattern/PATTERN-run-fsm.md`, the `pushResetRecords` comment and the summary's Change, Design decisions and Tests sections agree with the new code.

Tool calls (2 of 6):
- `grep` of `internal/webstercli/doc.go` and `sed -n` of `pattern/PATTERN-run-fsm.md` lines 28-50: my context held only a few matched lines of the pattern and none of `doc.go`.
- `Read` of `internal/webstercli/doc.go` lines 22-43: the full Reset section was not in my context.
