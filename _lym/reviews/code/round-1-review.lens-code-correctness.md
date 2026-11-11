```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff.

**Per-function list**
- `supersedeLocked` (`verbs.go`): empty topic is a no-op. A same-sender, same-receiver, same-topic unread message is returned `superseded` and its reply wait is answered. The other-sender, other-topic, read and reserved cases stay. Correct.
- `storePost` (`verbs.go:238-268`): a failing `supersedeLocked` append or `addWaiterLocked` after a successful `appendPostedLocked` returns an error with the post already journaled. Finding 2.
- `Store.DropRun` (`verbs.go`): a non-program sender, an empty run, no match, a read message and a reserved message each behave as specified. Matches are sorted by id. Correct.
- `DropRun` in `server.go` and `protocol.go` (the verb's dispatch and request/response fields): the response carries `Dropped`. Correct.
- `fold.go` and `journal.go`: `Topic` and `Run` survive `applyPosted`, `applyCarried` and `carryRecords`. An unread carried message can still be superseded or dropped by tag. Correct.
- `GoClient.DropRun` (`goclient.go`): a missing journal directory returns `nil, nil` with no daemon started. An empty `journalDir` also reads as not-exist. Any other `Stat` error falls through to the call. An old daemon rejects the unknown verb, which comes back as an error and becomes a Warn in the dropper. Unknown JSON fields on `post` are ignored, so an old daemon silently drops the tags. Correct.
- `PostTagged` / `notifyPrime` / `postNotice` (`orchcli/notify.go`): the tags pass through both the strand-claim path and the program fallback. Correct.
- `NotifyParent` (`notify.go`): `Run` is `pair.WorktreeName`, which `lymcwd.go:176` sets to the base of the worktree root. That equals the slug for a pair. Correct.
- `hubDobbyDeps` `runNotices.Post` (`beamcli/dobbydeps.go`): topic `dobby-run-notice/<slug>`, run `<slug>`, signed as the dobby's scoped program. `requirePosted` keeps the retry when no orch is recorded. Correct, with finding 3.
- `deliverRunNotice` (`runnotice.go`): a nil `Post` returns false without a Warn. Production always sets `Post`, so it only affects tests. Correct.
- `recordPushFailure` / `PushNotice` / `pairSlug` (`failedpush.go`): the topic `push/<repo>/<branch>` is stable per branch. The code-branch case (`Repo == code && !Prime`) uses the base of `RepoPath`, which is the pair worktree. The records case uses the base of `CodeWorktree`, which `pairedCodeWorktree` fills for every records record, so the prime's own records branch likely gets a non-empty `Run`. Finding 4.
- `InstallPushNotifier` (`pushnotifier.go`): the route is the same as before, with tags added. Correct.
- `postRemoteTeardownNotice` (`remoteteardown.go`): the topic is `remote-teardown/<slug>` and there is no run tag, so teardown does not drop it. Intended.
- `RemovePair` (`teardown.go:475`): the drop runs only when `remove` and `clearStray` returned no error. It runs before the board settle, and a dropper error is only logged. Teardown paths that reach `RemovePair` (the `EndForCancel` route and `req.Local` included) all drop. A nil `t.dropRunMail` is not possible because `New` always sets it. Correct.
- `dropPairRunMail` (`pairteardown/runmail.go`): no installed dropper means nothing is dropped. An error produces a Warn. A count of 0 stays silent. Correct.
- `InstallRunMailDropper` (`orchcli/runmail.go`): `NewProgramClient` errors propagate to the Warn. Creel refuses a program claim from a non-orch strand pane, so a teardown run there silently leaves the mail. That is documented, but it is a gap for a stale-mail fix. Finding 5.

**Findings**
1. MEDIUM, `design`, `pattern/PATTERN-pair-remote.md:70-74`. The new bullet was inserted in the middle of the board-settle bullet. The orphaned lines "A `running` record with no live registered batten process means the run is gone. The composite keeps the claim and warns with `lym batten cancel <slug>`." now read as part of the run-mail-drop bullet. The board-claim rule is left incomplete. The bullet belongs after line 74, not before line 73. Evidence: file read lines 66-75.
2. LOW, `design`, `verbs.go:250-258`. When the supersede append fails after the new post is journaled, `storePost` returns an error for a post that was stored, and a caller that retries posts a duplicate. The supersede is a best-effort cleanup, so it should not fail the post. The other failures of the same locked callback after `appendPostedLocked` (`addWaiterLocked`, `deliverMailLocked`) behave the same way, so it is a minor inconsistency.
3. MEDIUM, `design`, `beamcli/dobbydeps.go` (topic per run) and `battencli/notices.go:31`. Every dobby notice of one run shares the topic `dobby-run-notice/<slug>`. A run-ended or landed notice from batten, tagged with the run, is dropped at the run's teardown when still unread. So an unread run notice (dead driver, then quiet) can be replaced by a later notice of a different condition, and a final notice is dropped on teardown. The goal asks that a changed state still reach the orch. A newer notice does reach it, but an unread older one of another condition is lost. I judged this acceptable and rate it MEDIUM because it is a deliberate tradeoff the record does not pin.
4. LOW, `design`, `failedpush.go:186-188`. A prime's records push failure has `CodeWorktree` set via `pairedCodeWorktree`, so `pairSlug` may return the prime's directory name as `Run`. That is harmless unless a pair is later torn down under the same slug. The doc says it answers empty for such a branch, and `pairSlug` does not check `Prime` on the records case.
5. LOW, `design`, `orchcli/runmail.go:17-20`. A teardown run from a non-orch strand's pane drops nothing, and only a Warn (or an error) says so. Stale mail then still reaches the orch. The limit is documented in the comment, and the fix would need creel to accept the claim.

**Tool calls**
1. `grep` of `CodeWorktree` / `WorktreeName` and a read of `storePost`: to see how `CodeWorktree` is set and the locked callback's failure order, which the manifest lacked.
