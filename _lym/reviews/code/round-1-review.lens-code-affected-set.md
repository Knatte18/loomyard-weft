```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens, covering what the push transfer/record split forces outside the lines the diff touches.

**Per-change set**

| Change | Set found | Covered? |
|---|---|---|
| Pushes go to the push URL and write the tracking ref and upstream afterwards under the lock | The `gitrepo` push methods; `fabricengine` callers `pushclassified.go`, `destroy.go`, `archive.go`, `recordsgit.go`; the docs in `gitrepo/doc.go`, `fabricengine/doc.go` and `originreconcile.go`; `pattern/PATTERN-remote-refs-lock.md` | Partly; see F2–F5 |
| Pinned CLI-bound method list | `cmd/lym/gitrepoboundary_test.go`; the three new helpers all call `runChecked` | Covered |
| Lock enforcement test | `cmd/lym/remoterefslock_test.go` covers fetches only. Nothing in the diff pins that a push writes no tracking ref outside `gitrepo`. | Covered by the existing guard plus the new integration test |
| The push reflog subject `update by push` is now written by lym | `fabricengine/reflog.go` reads it (`trackingReflog`, `classifyRewrite`, `holdBackRewound`) | Not covered; see F1 |
| `push.autoSetupRemote` dropped from production | `toolfloor/doc.go`, `boardengine/boardtest/sync_test.go`, and `GitError` fixtures in `gitexec`, `shedtransient` and `landingshed` tests | Partly; see F5 |
| `PATTERN.md` row | Net-unchanged against the base. The new rule lives in the `pattern/` background file. No declared PATTERN changes appear in my context. | Covered |
| Commit trailers | No `Pattern-Departure:` trailer in the range | Nothing to check |

**Findings**

**F1. BLOCKING, design.** `PATTERN-named-constants`: `internal/gitrepo/pushdestination.go:19` adds `trackingReflogMessage = "update by push"`. `internal/fabricengine/reflog.go:16` already holds the same literal as `pushReflogSubject`.
- `trackingReflog` (`reflog.go:54`) stops its walk at that subject, so the two literals are now a load-bearing contract. `gitrepo` writes the value and `fabricengine` reads it.
- Nothing ties them together. `fabricengine` already imports `gitrepo`, so one exported constant would serve both.
- The `reflog.go:15` comment still says git writes the subject, which is no longer true.

**F2. MEDIUM, design.** A remote change that landed is reported as a failure, and callers that act on the error lose the mutation record.
- `DeleteRemoteBranch` returns `(true, err)` when only the local recording step fails (`push.go:258-268`). `destroy.go:1523-1526` appends `KindRemoteBranchDeleted` only when `err == nil`, so a landed remote delete goes unrecorded.
- `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased` return a bare error after the remote already changed. `destroy.go:1516-1518` returns without recording, and `destroy.go:1547-1552` words it as "moving the remote task branch … failed", with no `KindRemoteBranchUpdated`.
- The two leased doc comments (`push.go:285`, `push.go:312`) do not state this partial-failure contract.

**F3. LOW, design.** A failure of `recordPush` after a landed push reaches `pushClassified` as an error no class names.
- Evidence: `pushclassified.go:135-138` runs `job.fail(PushFailureOther, …)` and `pushclassified.go:119` skips `landed`.
- The result is a failed-push record and a creel notice for a push origin already holds.

**F4. LOW, consistency.** Two comments that this diff outdates were left as they were.
- `gitrepo/remoterefslock.go:24` says a holder "sees no push or fetch of the repository land in between". A push transfer can now land, and only its tracking-ref write is excluded.
- `fabricengine/originreconcile.go:785-786` says an unreachable origin "blocks every fetch and push of both repositories". Now only fetches and push record steps wait.

**F5. LOW, consistency.** Two comments still treat `push.autoSetupRemote` as something lym relies on.
- `toolfloor/doc.go:23` lists the key as a git feature found.
- `boardengine/boardtest/sync_test.go:28` says a push does something "via push.autoSetupRemote".
- No production code sets the key any more (grep over non-test `.go` files shows only comments).

**Tool calls (3)**
1. A grep for `autoSetupRemote` and remote-refs-lock text in non-test `.go` and `.md` files. My context lacked a repo-wide sweep for stale mentions.
2. A read of `merge.go:330-360` for `ResolveSHA`. My context lacked its definition. It is a go-git revision resolve, so a raw sha source works for `PushCommit`.
3. A grep for `autoSetupRemote` in test files and for `update by push` in `.go` files. My context lacked the test fixtures and the reflog-subject sites.
