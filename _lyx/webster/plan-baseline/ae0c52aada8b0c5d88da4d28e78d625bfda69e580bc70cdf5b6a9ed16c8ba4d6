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
- `plan:internal/fabricengine#recordsConflictWayForward` -> `func recordsConflictWayForward(worktree, branch string, paths, commits []string, pushVerb string) string`
- `plan:internal/fabricengine#codeDivergedWayForward` -> `func codeDivergedWayForward(worktree, branch string, prime bool) string`
- `plan:internal/fabricengine#rewoundWayForward` -> `func rewoundWayForward(worktree, branch, originHead string, dropped []string) string`
- `plan:internal/fabricengine_test#TestFailedPushRecord` -> `func TestFailedPushRecord(t *testing.T)`

**Intent:** D10's failed-push record, the store every later push card writes through.
`lyxdirs` declares the directory name, and the record lives at `<HubScratchDir(hub)>/failed-push/`, machine-local under the board-level `.lyx`, per PATTERN-durable-vs-ephemeral-state and PATTERN-lyxdirs-single-declarer.
A new fabricengine file (`failedpush.go`) holds one JSON file per repo and branch (the file name derived from both, escaped so a slash in a branch name stays one file), written only through `internal/state` so an unknown key survives (PATTERN-tolerant-records).
`FailedPush` holds the repo (`code`, `records` or `bolt` side, `bolt` being the records repository's board branch, plus the repo path), for a `records` record the pair's code worktree (the weft sibling's path with `weftname.Suffix` trimmed from its directory name, which for the prime weft names the prime's own worktree), since every fabric verb refuses to run in a weft sibling, the branch, a `Prime` flag set for a `code` record of the hub prime's own branch, the class, git's last error line, the conflicting paths, the local commits and origin's head where the failure has them (a replay stop, a rewind), the first and last time seen, the count and the `notified` flag.

`RecordPushFailure` updates the record under the record's lock, creating it on the first failure and bumping count and last-seen on later ones.
While `notified` is unset and the class is not `deferred`, it calls the process's installed `PushNotifier` once with the hub path and a line naming the branch, the class, the error line and `FailedPushWayForward`, and sets `notified` only when the notifier reports the mail posted.
No installed notifier, or a notifier reporting not posted (no orch recorded), leaves `notified` unset, so the first failure after an orch is recorded mails; a notifier error is logged and leaves it unset.
`InstallPushNotifier` stores the notifier process-wide, as `pauserecord.InstallGate` stores the gate, and returns the restore.
`ClearPushFailure` removes the record and is a no-op when none exists; `ListFailedPushes` returns every record sorted by repo and branch, an empty directory reading as none.
`FailedPushWayForward` names, per side and class, a runnable allowed way forward (PATTERN-no-denied-recovery):
- `deferred` and `transport`: the side's push verb, `lyx fabric push` in the branch's worktree for a `code` record and in the record's code worktree for a `records` record, `lyx board sync` from the prime for a `bolt` record, since `lyx fabric push` pushes a pair and never Bolt, and `git push` in the prime worktree for a `Prime` record, since `PushPairAnchored` in the prime pushes its records side only and reports the code branch as `code_push_skipped`;
- `auth`: the remote's permission fix, then the same push verb;
- `not_fast_forward` on the `records` or `bolt` side: `recordsConflictWayForward` over the branch's worktree, its push verb the side's push verb with the directory it runs in;
- `not_fast_forward` on the `code` side: `codeDivergedWayForward` with the record's `Prime` flag, the shape commit-records' `pushFailureWayForward` gives a rejection;
- `gone`: the deleted branch, and that its board entry is neither done nor removed on origin, since an ended entry writes no record; the operator decides whether the run ends, `lyx fabric remove` of the pair the record's worktree belongs to, run from the prime, removes the pair and clears the record, and once origin's board shows the entry done or removed, `lyx cleanup --apply` tears it down instead.
`recordsConflictWayForward` names the conflicting paths and the local commits, that another machine wrote the same paths on the branch, and the two choices with their commands: keeping origin's side by `git reset --keep origin/<branch>` in the worktree, which drops the local commits, or keeping this machine's by `git merge origin/<branch>` there, settling the named paths and committing, then the side's push verb, whose push is then a fast-forward and meets no replay.
The git steps run in the branch's worktree, the weft sibling for a weft, and `pushVerb` carries the push step with the directory it runs in, for a weft `lyx fabric push` in the pair's code worktree, so the line names each command where it can run.
`codeDivergedWayForward` names `git merge origin/<branch>` in the task worktree, settling any conflict and committing, then `lyx fabric push`; with `prime` set it names the same merge in the prime worktree, then `git push` there, and never `lyx fabric push`, which leaves the prime's code branch unpushed.
No line of a `Prime` record names `lyx fabric push`; `git push` is outside PATTERN-no-denied-recovery's closed list.
`rewoundWayForward` names, for a task branch origin rewound, the dropped commits and the two choices D4 allows: `git reset --keep <origin head>` in the task worktree then `lyx fabric push`, which drops the local commits past origin's head, or `git push origin <branch>` there, a plain fast-forward since origin's head is an ancestor of the local tip, which restores the commits the other machine dropped and writes the push entry that ends the reflog test.
The three helpers are the one source of these lines for every stop card 5, 6, 9 and 15 names.
Any git command a way forward names must pass `internal/shellguard`'s rows for agents, and none names the removed `reset --hard` or `pull --rebase` recoveries.

The notifier is process-global state: `TestFailedPushRecord` and every other test that installs one runs without `t.Parallel`, naming the global, and its fake counts only calls carrying its own hub path, so a parallel test's recorded failure in another hub never reaches its counts.
`TestFailedPushRecord` (untagged, a temp hub dir, a fake notifier) has rows: first failure creates the record and mails once, the fake recording the line it receives, which equals the composed line naming the branch, the class, the error line and `FailedPushWayForward` of the record exactly; a second failure updates count and sends no mail; a notifier reporting not posted leaves `notified` unset and the next failure mails; a notifier returning an error leaves `notified` unset and the record written, and the next failure mails; `deferred` writes the record and never mails; `ClearPushFailure` removes it and a second clear is nil; `ListFailedPushes` over an absent directory is empty; an unknown key in a record file survives an update; one row per side and class pair a push writes asserts its exact way-forward line: `deferred` on `records` and `bolt`, `transport`, `auth` and `not_fast_forward` on each of `code`, `records` and `bolt`, and `gone` on `code` and `records`, each `records` line naming `lyx fabric push` in the pair's code worktree and, for `not_fast_forward`, the `git reset --keep` and `git merge` steps in the weft worktree, and a `records` record of the prime weft naming the push in the prime's worktree; and a `Prime` code record in each of `transport`, `auth` and `not_fast_forward`, asserting its line names `git push` in the prime and no `lyx fabric push`.

**Commit:** 1: fabricengine: the failed-push record and its notice
