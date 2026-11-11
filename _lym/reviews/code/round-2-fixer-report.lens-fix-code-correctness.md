```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 change, the new `TestAnswerParentAsk_FailedWriteAtDeadline` in `internal/burlerengine/parentreview_test.go`.

No findings.

I ran the test step by step:
- **Setup:** `recordParentAsks` creates the open ask with deadline `parentAskBase`. `os.Chmod(files.Dir, 0o500)` then makes the record directory read-only.
- **Lock:** the lock file is not in `files.Dir`. `ParentReviewFilesOf` sets `Lock` to `lock.TrackedLockPath(anchor, name)`, which lives in a separate locks directory. Lock acquisition therefore succeeds, and the failure comes from the record write, which is the branch under test.
- **Past-deadline branch:** `now` is `parentAskBase+1m`, so the branch sets the ask to timeout and returns `record, nil`. The write then fails, which is an unrelated write error and not `errParentAskUnchanged`.
- **Result:** `AnswerParentAsk` now returns that error wrapped (`parentreview.go:237`) ahead of the refusal. The test sees a non-nil error and `errors.As` fails, and the record on disk still reads open, so all three assertions hold. The gate output confirms `--- PASS: TestAnswerParentAsk_FailedWriteAtDeadline`.
- **Test fails with the defect:** before `32bd1767`, `if refusal != nil { return refusal }` came first. With a failed write, `refusal` was still set, so the old code returns a `*ParentAnswerRefusal`. The `errors.As` check would then fire and the test would fail, so it pins the F8 ordering.
- **Cleanup:** `t.TempDir` is registered inside `recordParentAsks`, before the test's `t.Cleanup` that restores `0o700`. Cleanups run last-in first-out, so permissions are restored before the temp directory is removed.
- **Skips and parallelism:** the Windows/root skip comes before any chmod. `chmod` touches only the test's own directory, so `t.Parallel()` is safe.
- **Callers:** the change is test-only, so no signature or behaviour changed for callers.

Tool calls (1 of 6):
1. `git show 32bd1767`, the lock-path lookup (`TrackedLockPath`), and `writeJSONUnlocked`. I needed the exact pre-fix ordering to confirm the test fails with the defect. I also needed to confirm the lock directory is separate from `files.Dir`. My context held neither.
