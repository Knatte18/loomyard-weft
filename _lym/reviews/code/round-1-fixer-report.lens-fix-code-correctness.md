```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed.

**MEDIUM**: `_lym/landing/summary.md`, `## Merge danger`: "The detached push already sent that commit there, so the synchronous push puts nothing new on origin." The code does not guarantee this.
- `Fabric.commit` only starts that push, fire-and-forget, and discards the start error: `_ = spawnDetachedPushFn(pushRequest)` (`internal/fabricengine/commit.go:169-172`, whose comment says a child that cannot start leaves the commits local).
- When the synchronous push runs, the detached child may not have pushed yet, may have failed, or may never have started. The synchronous push can then be the one that first puts the commit on origin.
- The review's own Rejected entry says the detached push "can carry the same commit to origin", not that it already has.
- The `two-way` door rests on this sentence. A true version would say that the detached push already targets the same commit, so the synchronous push sends nothing that push would not.

No findings in the Go changes:
- **Hand-over branch in `reset.go`:** a nil push error still refuses with the sync message. An ordinary push error keeps that message and adds `push_error`. An error wrapping `ErrHandedOver` replaces the message and keeps `push_error`. A nil `pushRecords`, `SkipGit` or `SkipPush` gives nil. No caller or signature changed.
- **`BeforeTheCommitLandsPushesNothing`:** the second open is the one inside `fabricSync`. If the `!committed` guard were removed, the stub's error would appear in `push_error` and the test would fail.
- **`AfterTheCommitLandsAHandOverReplacesTheWayForward`:** without the new branch, the message holds the sync way forward and lacks the hand-over text, so the test would fail.
- **`squatCorrespondenceIndex`:** it does the same as the inline code it replaced.

I made no further tool calls.
