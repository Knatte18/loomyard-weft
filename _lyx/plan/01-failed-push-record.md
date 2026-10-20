# Card 1 — failed-push-record

**Create:**
- `plan:internal/lyxdirs#FailedPushDirName` -> `const FailedPushDirName = "failed-push"`
- `plan:internal/fabricengine#PushFailureClass` -> `type PushFailureClass string`
- `plan:internal/fabricengine#PushFailureNotFastForward` -> `const PushFailureNotFastForward PushFailureClass = "not_fast_forward"`
- `plan:internal/fabricengine#PushFailureTransport` -> `const PushFailureTransport PushFailureClass = "transport"`
- `plan:internal/fabricengine#PushFailureAuth` -> `const PushFailureAuth PushFailureClass = "auth"`
- `plan:internal/fabricengine#PushFailureGone` -> `const PushFailureGone PushFailureClass = "gone"`
- `plan:internal/fabricengine#PushFailureDeferred` -> `const PushFailureDeferred PushFailureClass = "deferred"`
- `plan:internal/fabricengine#FailedPush` -> `type FailedPush struct`
- `plan:internal/fabricengine#PushNotifier` -> `type PushNotifier func(hubPath, line string) (posted bool, err error)`
- `plan:internal/fabricengine#InstallPushNotifier` -> `func InstallPushNotifier(n PushNotifier) (restore func())`
- `plan:internal/fabricengine#RecordPushFailure` -> `func RecordPushFailure(hubPath string, f FailedPush, now time.Time) error`
- `plan:internal/fabricengine#ClearPushFailure` -> `func ClearPushFailure(hubPath, repo, branch string) error`
- `plan:internal/fabricengine#ListFailedPushes` -> `func ListFailedPushes(hubPath string) ([]FailedPush, error)`
- `plan:internal/fabricengine#FailedPushWayForward` -> `func FailedPushWayForward(f FailedPush) string`
- `plan:internal/fabricengine_test#TestFailedPushRecord` -> `func TestFailedPushRecord(t *testing.T)`

**Intent:** D10's failed-push record, the store every later push card writes through.
`lyxdirs` declares the directory name, and the record lives at `<HubScratchDir(hub)>/failed-push/`, machine-local under the board-level `.lyx`, per PATTERN-durable-vs-ephemeral-state and PATTERN-lyxdirs-single-declarer.
A new fabricengine file (`failedpush.go`) holds one JSON file per repo and branch (the file name derived from both, escaped so a slash in a branch name stays one file), written only through `internal/state` so an unknown key survives (PATTERN-tolerant-records).
`FailedPush` holds the repo (`code` or `records` side plus the repo path), the branch, the class, git's last error line, the first and last time seen, the count and the `notified` flag.

`RecordPushFailure` updates the record under the record's lock, creating it on the first failure and bumping count and last-seen on later ones.
While `notified` is unset and the class is not `deferred`, it calls the process's installed `PushNotifier` once with the hub path and a line naming the branch, the class, the error line and `FailedPushWayForward`, and sets `notified` only when the notifier reports the mail posted.
No installed notifier, or a notifier reporting not posted (no orch recorded), leaves `notified` unset, so the first failure after an orch is recorded mails; a notifier error is logged and leaves it unset.
`InstallPushNotifier` stores the notifier process-wide, as `pauserecord.InstallGate` stores the gate, and returns the restore.
`ClearPushFailure` removes the record and is a no-op when none exists; `ListFailedPushes` returns every record sorted by repo and branch, an empty directory reading as none.
`FailedPushWayForward` names, per class, an allowed way forward: `lyx fabric push` in the branch's worktree for `deferred` and `transport`, the remote's permission fix then `lyx fabric push` for `auth`, `lyx cleanup --apply` for `gone`, and for `not_fast_forward` a line saying another machine wrote the same paths on the branch, naming the paths and the local commits, and that the operator decides which machine keeps the run before the push is retried with `lyx fabric push` (PATTERN-no-denied-recovery); any git command a way forward names must pass `internal/shellguard`'s rows for agents, and none names the removed `reset --hard` or `pull --rebase` recoveries.

The notifier is process-global state: `TestFailedPushRecord` and every other test that installs one runs without `t.Parallel`, naming the global, and its fake counts only calls carrying its own hub path, so a parallel test's recorded failure in another hub never reaches its counts.
`TestFailedPushRecord` (untagged, a temp hub dir, a fake notifier) has rows: first failure creates the record and mails once; a second failure updates count and sends no mail; a notifier reporting not posted leaves `notified` unset and the next failure mails; a notifier returning an error leaves `notified` unset and the record written, and the next failure mails; `deferred` writes the record and never mails; `ClearPushFailure` removes it and a second clear is nil; `ListFailedPushes` over an absent directory is empty; an unknown key in a record file survives an update; each class's way forward is the exact line.

**Commit:** 1: fabricengine: the failed-push record and its notice
