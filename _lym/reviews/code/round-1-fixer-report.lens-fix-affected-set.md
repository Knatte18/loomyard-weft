```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens — check that each finding premised on a set (gate sites, lock-before-gate, doc mentions) is covered by the fix in full.

**MEDIUM — `TakeOverRun` still waits at the gate while it holds teardown locks (F1 set)**
- **Changed text:** `takeoverBranch.land`'s `Admit` in `takeoverwrite.go:173`, and the `doc.go:1245` paragraph.
- **Problem:** F1's premise is that a pause must not hold a waiter that keeps the pair's teardown lock. The fix moved the gate out of that case only for `PushPairAnchored`.
- **Evidence:** `takeoverwrite.go:105-118`, as read earlier, takes the write teardown lock of the pair and of its parent with `lock.AcquireWriteLockWithin(TeardownLockPath…)`. `TakeOverRun` then runs fetches and archives, and only afterwards calls `land`, which is the first `Admit`. A pause that covers the process holds `TakeOverRun` at that `Admit` with both teardown locks held, so other pushes and `Remove` calls on those pairs still fail or wait.
- **Doc claim:** The new text in `doc.go:1246-1248` says a pause "holds a pusher only while it holds no lock".
- **Mismatch:** The new `TakeOverRun` row cannot catch this. Its probe checks only the push locks.
- **Fix needed:** Either admit before the teardown locks in `TakeOverRun`, or say in the summary and `doc.go` that `TakeOverRun` is the accepted exception.

**LOW — other teardown-lock takers that reach a gate are unchecked (F1 set)**
- **Candidates:** `ownerarchive.go:75`, `handover.go:71`, `handover.go:484`, `remoteteardown.go:347` and `remoteteardown.go:398` take the teardown lock. I did not read them.
- **What to check:** Whether any of them calls `coalesceFlagged`, a Bolt pull or another `Admit` site while the lock is held. If so, the same gap applies.
- **Why it matters:** The `doc.go` sentence "A pause therefore holds a pusher only while it holds no lock" is stated without that exception.

**Checked, no finding**
- **Callers of `pullFirst`:** only `Bolt.Pull` (`bolt.go:284`) and `PullThenCommitWritten` (`bolt.go:226`), and both now admit first, so removing the inner `Admit` is safe.
- **Callers of `PushPairAnchored`:** `fabriccli/weft_verbs.go:301` and `pairteardown/handover.go:79` both go through the new outer `Admit`. `PushRun` holds no lock before the call.
- **Gate-before-lock prose:** every sentence that says it passes the gate before the lock matches the code, and none still names `pullFirst`. The only list that goes stale is the `doc.go` paragraph, which was updated.
- **Pause-gate doc coverage:** the `doc.go` pause paragraph, the `pauserecord/doc.go` hold paragraph (F9) and the `coalesce.go:19` sentence say the same thing. `coalesce.go:19` still reads "before it takes its first lock", which is now true for the callers named.
- **Skipped docs (F10–F12):** the tender mentions are in `tenderengine/doc.go`, `report.go`, `resume.go` and `pause.go`, plus `tendercli/cli.go` and `docs/overview.md`, all updated. The `pause.go` `PauseReport.Skipped` comment already mentions the pair with no reed session.

**Tool calls**
1. A grep for the lock takers, the callers of `pullFirst`, `PushPairAnchored` and `coalesceFlagged`, the gate prose, the skipped docs and the remote-refs wording. It reran F1, F3, F5, F9, F10, F11 and F12's searches, which my context lacked.
2. `sed` on `pairteardown/handover.go` plus a grep for `TeardownLockPath(`. It checked the `PushPairAnchored` callers and listed the other teardown-lock takers.
