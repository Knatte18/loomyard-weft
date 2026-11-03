```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F4 change set for stale siblings and twins left unchanged.

**MEDIUM** — `remoteTeardown.go`, `runRemoteTeardown` and `remoteOutcome.weftRootErr`: the records-repository lookup failure is now stored outside `out.err`, but the in-process consumers read only `out.err`, so it is dropped.
- Before the fix, `runRemoteTeardown` stored this failure in `out.err`. The in-process callers of `hold.finish(job)` read `out.err` and no other error field.
- `remove.go:402` returns `res, out.err`. A removal whose lookup failed now returns a nil error.
- `cleanup.go:321` sets `entry.Error` only when `out.err != nil`. A lookup failure now leaves the entry looking clean.
- The `taskbranchdiscard.go:66-67` `out.ran` branch may read `out.err` the same way. I did not open it.
- The in-process hold is the path every test binary takes, because `holdTeardown` checks `testing.Testing()`, and it is also the path for `handOff` false.
- Only `failedRemoteSteps` reads `weftRootErr`, and it runs only in the child via `noticeRemoteTeardown`.
- Nothing tests the in-process consequence.
- Twin to update: those three callers need to treat `weftRootErr` as a failure, or `runRemoteTeardown` needs to expose one combined error.

No other findings. I checked the following and they agree with the change:
- The `RunTeardownChild` doc still holds ("logged and mailed… the operation and the way forward").
- The `remoteteardown.go` file header is unchanged and still accurate.
- `doc.go:612` and `remove.go:159` do not mention the changed notice or signal behaviour.
- The `warnWhenEnded` doc matches its new body.
- The two other callers of `noticeRemoteTeardown`, both in `RunTeardownChild`, pass the new argument.
- The `unknownKindWayForward` text is shared by the unreadable-job notice and the ended notice.
- `discardRerunWayForward` is used in `failedRemoteSteps`, and its text matches `discardAgain` in the test.

Tool calls:
1. grep of docs and Go files for twins mentioning the teardown child, signal, notice or `spawnTeardownHold`: to find cross-references outside the changed files.
2. `git show HEAD --stat` plus grep for "weft repo root": to confirm the changed-file set and see sibling uses of the message.
3. grep for `.finish(`, `remoteOutcome`, `.ran` and related names in non-test code: to find the consumers of `remoteOutcome`.
4. Read of `remove.go` lines 366–416: to see how the in-process outcome is consumed.
5. Read of `cleanup.go` lines 316–346: to see how the in-process outcome is consumed there.
