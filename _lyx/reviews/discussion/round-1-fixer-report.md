# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING), fixed as proposed: premise confirmed in `internal/landingshed/verifygate.go`, where `recordFailure` writes before `verdict` returns Stuck, and `verifytree.PublishFailure` has no time field.
  D5 now has Publish write `written_at` into the record and matches it against the halting Publish attempt, after the child's last history entry before it and no later than the halting Publish entry, never against the blocked state's start; a same-HEAD record from an earlier halt is stale.
  D5's bound and the stale-record criterion updated.
- F2 (MEDIUM), fixed as proposed: `hubWriteRefusal`'s doc in `internal/configcli/hubwide.go` says it "guards a mistaken write, not a determined agent"; `update-plugins.sh` execs `go run ./tools/deploy`, which moves `prod`.
  D2 now calls the pair-write refusal a mistaken-write guard, says the command runs landed-tip code and may push `prod`, names PR-Gate approval as the trust anchor, and its rationale and bound match.
- F3 (MEDIUM), fixed as proposed: `rebaseRetryTriggers` in `internal/gitrepo/push.go` holds `rejected`, used by `pushWithRebaseRetry` and `PushRebaseFree`.
  D6 names the retried primitives (`Push`, `PushCoalesced`, `PushRebaseFree`, Add's branch pushes, the archive tag push in `archive.go`), excludes `UpdateRemoteBranchLeased` and both deletes, orders the check before the rebase trigger and the `ErrPushRejected` mapping, and says an exhausted retry returns a plain push failure.
  The push-retry criterion updated.
- F4 (MEDIUM), fixed as proposed: D5's goto and resume retries get a time cap (Plan picks the value), after which the blocked notice goes out with the last refusal; a busy pane gets the awaiting arm's driver-busy notice (`noticeDriverBusy`).
  The bound and the retry criterion updated.
- F5 (MEDIUM), fixed as proposed: D4 defines an episode as one entry of the row from its predecessor's Done or from a halt at that row; a restart of a batten that died mid-row continues the episode, a `lyx batten run` after a halt starts a new one.
  The deploy-failure criterion now separates the two.
- F6 (LOW), fixed as proposed: Goal's wake list adds land and teardown failures and the ready notice, temporary until `orch-split`.
- F7 (LOW), fixed as proposed (second option): D6's rationale says "the common cause", and its bound states a permanent remote "cannot lock ref" is retried to the attempt cap under the caller's lock.
  Narrowing the match was not chosen: the race and the conflict share the reason text, and no stable distinguishing form was found.
- F8 (LOW), fixed as proposed: D1 step 1 waits on a dirty prime only when the pull will run; the dirty-prime criterion adds the skipped-landing case.
- `support-log.md`'s `## Review rounds` records this round.

## Disputed

None.

## Deferred

None.
