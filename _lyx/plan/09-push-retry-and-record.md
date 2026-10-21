# Card 9 — push-retry-and-record

**Create:**
- `plan:internal/fabricengine#ErrBranchGone` -> `var ErrBranchGone error`
- `plan:internal/fabricengine#pushClassified` -> `func pushClassified(rec *Mutations, hubPath, side, path string, opts SyncOptions, retryRecords func() (deferred bool, err error)) error`
- `plan:internal/fabricengine#branchGoneOnOrigin` -> `func branchGoneOnOrigin(path string) (gone bool, err error)`
- `plan:internal/fabricengine#goneEntryEnded` -> `func goneEntryEnded(hubPath, branch string) (bool, error)`
- `plan:internal/fabricengine#retryWeftPush` -> `func retryWeftPush(rec *Mutations, hubPath, weftPath string) (deferred bool, err error)`
- `plan:internal/fabricengine#retryBoltPush` -> `func retryBoltPush(rec *Mutations, boardDir string) (deferred bool, err error)`
- `plan:internal/fabricengine#PushFailureRewound` -> `const PushFailureRewound PushFailureClass = "rewound"`
- `plan:internal/fabricengine#PushFailedError` -> `type PushFailedError struct`
- `plan:internal/fabricengine#PushFailedError.Error` -> `func (e *PushFailedError) Error() string`
- `plan:internal/fabricengine#PushFailedError.Unwrap` -> `func (e *PushFailedError) Unwrap() error`
- `plan:internal/fabricengine_test#TestPushFailureClasses` -> `func TestPushFailureClasses(t *testing.T)`

**Edit:**
- `internal/fabricengine#SyncOptions`
- `internal/landingshed#TestMergeIn_SyncParent`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#PushAnchored`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#pushRebaseFreeRetryingOnce`
- `internal/fabricengine#Bolt.PushRecorded`
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine/doc.go`
- `PATTERN.md`
- `internal/fabricengine_test#TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejectedUnwrapped`
- `internal/fabricengine_test#TestCoalescePushBothAt_DivergedWarpRemote_ReturnsNilWithoutSpinning`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`
- `internal/fabricengine#PushWarpAt`
- `internal/fabricengine#PushWarpRebaseFreeAt`
- `internal/fabricengine#FailedPushWayForward`
- `internal/orchcli#writeHubRules`
- `internal/orchcli#TestWriteHubRules`
- `internal/fabriccli#reconcileFailureWayForward`
- `internal/fabriccli#hubConfigFailure` -> `func hubConfigFailure(detail string, cause error, boardDir string) error`
- `internal/fabriccli#warpBindingFailure` -> `func warpBindingFailure(detail string, cause error, boardDir string) error`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli_test#TestRunCLI_ReconcileBoardFailureFailsTheVerb`
- `internal/loomcli#commitRecordsVerb`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestPublish_StuckBeforePullRequest`
- `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes`

**Delete:**
- `internal/fabricengine#pushRebaseFreeLogged`
- `internal/fabricengine#unpushedOutcome`
- `internal/fabricengine#pushWeftAt`
- `internal/fabricengine#Bolt.dropSeedCommitsAfterConflict`

**Uses:**
- `internal/fabricengine#RecordPushFailure`
- `internal/fabricengine#ClearPushFailure`
- `plan:internal/fabricengine#replayWeft`
- `plan:internal/fabricengine#replayBolt`
- `plan:internal/fabricengine#readEndedSet`
- `plan:internal/gitrepo#ErrPushAuthRefused`
- `plan:internal/gitrepo#PushBackoffDelays`
- `internal/fabricengine#PrimeName`
- `plan:internal/fabricengine#trackingReflog`
- `plan:internal/fabricengine#classifyRewrite`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D10: every push fabricengine runs classifies a failure, recovers where it safely can, and otherwise writes the failed-push record and its one notice; D5's replay replaces `gitrepo`'s pull-rebase on every fabric push path a production caller reaches.
`Fabric.PushWeft` and `PushWarpAt` stay on `gitrepo.PushCoalesced`, whose rejected push runs `git pull --rebase`, because no production path reaches either: only tests call them.

`pushClassified` is the one classifier the detached coalesced push (`coalescePushSides`), `PushAnchored`, `PushPairAnchored`, `PushWarpRebaseFreeAt` (reached through `Fabric.PushBranch` by Publish's task-branch push and Finalize's parent push, on the code side) and Bolt's pushes (`Bolt.PushRecorded`, reached by `Bolt.Push` and Bolt's syncs) run per side:
- Before pushing a weft or a task branch with an upstream it runs `branchGoneOnOrigin`, one `git ls-remote origin` of that branch; Bolt skips it, since Bolt's branch is never deleted.
  A gone branch is not pushed, so a parked push never recreates a branch landed or deleted elsewhere: `goneEntryEnded` fetches origin's Bolt branch (one fetch, the gone path's only Bolt refresh) and reads the ended set by D3 step 2's rule; an ended entry writes no record and is left to D6's teardown.
  A branch gone with its entry not ended, and a transport failure of the ls-remote or of the Bolt fetch, are retried by one backoff (the delays `gitrepo.PushBackoffDelays()` returns, slept through `SyncOptions`' new `BackoffSleep func(time.Duration)` field of the caller's opts, nil sleeping those delays, per PATTERN-test-speed), re-running the ls-remote, the Bolt fetch and the judgment; at its end a branch still gone with its entry neither done nor removed writes a `gone` record, and a transport failure still standing writes a `transport` record.
- A task branch's push is not run when `classifyRewrite` over the origin head that ls-remote returned and `trackingReflog` reads the branch as **rewound**: origin's head is then an ancestor of the local tip, so a plain push would fast-forward origin and restore the commits another machine's leased rewind dropped (D4's bound, and the criterion that its push does not restore them).
  It writes a `rewound` record carrying origin's head and the dropped commits, which mails, and `FailedPushWayForward` gains the class, naming card 1's `rewoundWayForward`; the branch's next push after the rewind is settled lands and removes the record.
  `lyx fabric push` stays refused until the rewind is settled, by `git reset --keep <origin head>` or by the raw `git push origin <branch>` that writes the push entry ending the reflog test.
  An `ours` merge of `origin/<branch>` is not offered: origin's head is already in the tip's history, so the merge changes nothing and the hold-back reads the tip as still rewound.
  Bound: only a task branch read as rewound is held back; a weft, Bolt and the prime's code branch are never judged so.
- A code-side push is a task branch's exactly when the worktree at `path` is not the hub's prime: `path` is compared with the prime's worktree path under `hubPath`, the prime resolved as `IsPrimeWorktree` resolves it, so Finalize's push of a non-prime parent pair is judged as a task branch's and the prime's own code branch is not.
  The gone check and the rewound hold-back read this predicate, and it sets `FailedPush.Prime` on a code-side record, which card 1's way forward reads.
- A records-side rejection (`gitrepo.ErrPushRejected`) calls the side's retry once: `retryWeftPush` fetches the weft branch and origin's Bolt branch, try-acquires the weft write lock while holding the push lock, reads the ended set from the fetched Bolt tip and calls `replayWeft`, then pushes again; `retryBoltPush`, holding `board.push.lock`, try-acquires `board.lock`, then takes `.gitrepo-push.lock`, fetches, reads the ended set and calls `replayBolt`, then pushes again.
  A held write lock or `ErrReplayDirty` is a deferral: the holder writes no pending flag of its side and ends its loop, leaving the commits to the next commit's push, and writes a `deferred` record (no mail), which the next landed push removes.
  A replay that stops (`ErrReplayConflict`) or a second rejection is a failed push of class `not_fast_forward`, its record carrying the replay's conflicting paths and the local commits, and the returned error still wraps `gitrepo.ErrPushRejected`, so `IsPushRejected` callers keep working.
- A code-side rejection is a failed push of class `not_fast_forward` at once; `pushRebaseFreeRetryingOnce` keeps only its stale-tracking-ref retry, now for the code side alone.
- A `gitexec.IsTransportFailure` error of the push call itself is class `transport` at once, since `gitrepo.PushWithRetry` retries only a `(failed)` or `cannot lock ref` refusal; an `ErrPushRetriesExhausted` error is class `transport` after that existing backoff; `gitrepo.ErrPushAuthRefused` is class `auth` at once, with no retry.
- The side passed to `pushClassified` and written on the record is `code`, `records` (a weft) or `bolt`, which `FailedPushWayForward` reads to name the side's push verb.
- A push that lands calls `ClearPushFailure` for its repo and branch.
- A failed push that `pushClassified` records returns a `*PushFailedError` holding the record (`FailedPush`, its class included) and the push's own error, which `Unwrap` returns, so `errors.Is` against `gitrepo.ErrPushRejected` or `ErrPushAuthRefused` and `IsPushRejected` see through it, and a caller reads the class with `errors.As`.
  `PushFailedError.Error()` is the push's own error followed by `; way forward: ` and `FailedPushWayForward` of the record, so every caller that prints the error carries its class's line; a caller that appends a way forward of its own appends it only when some failure in the error, each member of a joined error read through `Unwrap() []error`, is not a `*PushFailedError`, so each failed side's line appears once, also when `PushPairAnchored` returns the `errors.Join` of two sides' failures, and a side that failed before its push was classified still gets the generic line.
`coalescePushSides` keeps returning nil for a failed push, as the detached child always did, but the failure is now recorded rather than only warned; `PushAnchored`, `PushPairAnchored` and `PushWarpRebaseFreeAt` return the error as before and record it too, so Publish's `push failed: <error>` Stuck keeps its code and prints the class's line through the error's text for every class but `not_fast_forward`.
A `not_fast_forward` `*PushFailedError` wraps `gitrepo.ErrPushRejected`, so Publish keeps `pushRejectedReason` for it, whose remote-only-commits detail is richer than the class line, and appends nothing to it.
Finalize's `pushParent` Stuck prints the error the same way and drops its trailing `push it by hand` when `errors.As` finds a `*PushFailedError`, since a plain push is what a `rewound` hold-back of a non-prime parent pair refuses; `TestFinalize_MergeMarkPushOrderAndVerdict` gains rows whose push returns a `*PushFailedError` of class `rewound`, of class `not_fast_forward` with `Prime` set and of class `auth`, each reason ending in `FailedPushWayForward`'s exact line for its record with no `push it by hand`.
`TestPublish_StuckBeforePullRequest` gains a row for each of `gone`, `rewound`, `auth` and `transport`, its Stuck reason `push failed: ` plus the error text ending in the class's exact line, and a `not_fast_forward` row whose reason is `pushRejectedReason`'s text with no second way-forward line.
`loomcli`'s `commitRecordsVerb` prints `err.Error()` unchanged and appends `; way forward: ` plus `pushFailureWayForward`'s text only when some failure in the error is not a `*fabricengine.PushFailedError`, by the rule above, since that text's `transient, run lyx fabric push` answer would repeat the stop for a `gone` branch (never pushed), a `rewound` task branch (held back) and an `auth` refusal.
`Bolt.PushRecorded` pushes with `PushRebaseFree` instead of the pull-rebasing `pushWeftAt`, and its seed-commit recovery becomes `retryBoltPush`'s replay, so `dropSeedCommitsAfterConflict`, `pushWeftAt`, `pushRebaseFreeLogged` and `unpushedOutcome` are deleted with their callers here, and `PushWarpAt`'s doc comment and `spawn.go`'s file header, which describe it as the sibling of `pushWeftAt` with matching gating, are reworded to stand alone.
With Bolt's push no longer rebasing, `gitrepo.ErrPullRebaseFailed` never reaches `orchcli`'s `writeHubRules` from `bolt.Push()`: its `diverged` branch and the way forward naming `git pull --rebase` are dropped from both its `bolt.Push` refusals, the one before the write and the one after the commit: each prints `bolt.Push`'s error unchanged, which for a `*PushFailedError` already ends with that record's way forward (`FailedPushWayForward`), followed by the re-run of `lyx orch rules`, and keeps its own generic way forward (`re-run ... once Bolt pushes` before the write, `the next run of ... pushes it` after the commit) only when `bolt.Push`'s error is not a `*PushFailedError`; and `TestWriteHubRules`' fake push returns a `*PushFailedError` of class `not_fast_forward` instead of `ErrPullRebaseFailed`, its rows asserting the conflict's way forward in place of `git pull --rebase`.
`fabriccli`'s `reconcileFailureWayForward` drops its `git pull --rebase` clause for a rejected board push and names the failed-push record `lyx fabric status` prints, with `lyx board sync` once its way forward is followed.
`hubConfigFailure` and `warpBindingFailure` gain a `cause error` parameter beside the detail, which their call sites in `runReconcile` pass (the commit or push error each detail is built from), and append the line only when `cause` is not a `*PushFailedError`, by the rule `commitRecordsVerb` follows, since that error's text already ends in its class's line.
`TestRunCLI_ReconcileBoardFailureFailsTheVerb` gains a row whose board origin URL is `http://127.0.0.1:1/x` with the binding unbound, asserting the binding push's `transport` line appears exactly once and the reconcile line not at all, and its rejected-commit rows assert the reconcile line exactly once.
The hub is the parent of the pushed worktree, so the detached child needs no new flag.
`doc.go` replaces the push-retry and seed-drop paragraphs with this classification, and `PATTERN.md` gains the `PATTERN-push-failure-visible` entry line in its "Fabric and git" section, worded exactly as the overview's PATTERN changes bullet with `(test)`, within the entry line cap.
Bound: the retry replays only records branches, once per push, never waits on the write lock; a deferral costs at most one rejected push per commit; the notice is one posted mail per branch per episode.

`TestPushFailureClasses` (integration, `hubforge` hub, a second clone of each bare as the other machine, `boardkit.InstallReplay` called first, a fake notifier installed through `InstallPushNotifier` counting only its own hub's calls, the backoff's sleep set through `opts.BackoffSleep` of each row's push; run without `t.Parallel`, naming the process-global notifier) is the enforcing test.
Its transport input is the pushed worktree's `origin` URL set to `http://127.0.0.1:1/x`, which git refuses with `connection refused`, a phrase `gitexec.IsTransportFailure` matches; the gone-path rows restore the URL inside the `opts.BackoffSleep` they pass where a row needs a later attempt to answer.
Rows: a detached weft push rejected as not fast-forward is fetched, replayed and pushed; with the weft write lock held it defers, writes a `deferred` record with no mail and sets no pending flag, and the next commit's push lands it and removes the record; a detached weft push whose replay meets a both-sides `status.json` change writes a `not_fast_forward` record and mails; a records retry whose second push is rejected again (the other machine pushes once more inside the retry, through a pre-receive hook on the records bare that admits one push) writes a `not_fast_forward` record; two concurrent failures of one branch under a 403 hook write one record whose count is two and post one mail, and a landed push removes it; a push to the refused URL writes a `transport` record; a parked push of a branch gone on origin does not recreate it, and one of an entry ended on origin writes no record; a parked push whose gone-check ls-remote meets the refused URL, which the row's `opts.BackoffSleep` restores, records nothing and then pushes, and under a sleep that restores nothing records nothing until the backoff ends and then a `transport` record; a parked weft push of a run done on origin since this machine's last Bolt fetch writes no record; a weft push whose branch origin deleted before its Bolt shows done backs off, and the done mark arriving during the injected sleep ends it with no record; a prime-weft push rejected right after this machine's Finalize pushed `done` keeps the landing's commits; a Bolt push rejected over an unrelated origin entry is replayed and lands; a Bolt push rejected while `board.lock` is held defers with a `deferred` record; a task branch rewound on origin by a leased update, pushed from this machine with the dropped commits still local, is not pushed, origin's head stays where the rewind put it and a `rewound` record names the rewound way forward; the same rewound task branch pushed through `Fabric.PushBranch`, Publish's path, is not pushed either, writes a `rewound` record, and returns an error whose text ends in `rewoundWayForward`'s exact line; the same rewound task branch settled by `git reset --keep <origin head>`, and in a second row by the raw `git push origin <branch>`, then pushed by `lyx fabric push`, lands, removes the `rewound` record, and `classifyRewrite` over `trackingReflog` no longer reads the branch rewound; the prime's code branch pushed after an origin rewind of it is pushed and records nothing; a code-side push from the hub's prime worktree through `Fabric.PushBranch`, rejected because the other machine pushed the prime's branch first, writes a `not_fast_forward` record with `Prime` set and returns an error whose text ends in the prime form of `codeDivergedWayForward` (`git push` in the prime, no `lyx fabric push`); the same push under a 403 hook writes an `auth` record with `Prime` set whose line names `git push` in the prime.
`TestCommitRecordsVerb_ErrorEnvelopes` gains rows for a `*PushFailedError` of class `gone`, `rewound` and `auth`, each asserting the envelope's exact way-forward line and that it appears exactly once, and a row whose push fails on both sides (an `errors.Join` of a `gone` and an `auth` `*PushFailedError`) asserting each side's line appears exactly once, and a row joining a `gone` `*PushFailedError` with an unclassified error asserting the `gone` line once and the generic way forward once.
The code-side rejection's record is asserted in the changed coalesce test below, not again here.
The func field makes `SyncOptions` non-comparable, so `TestMergeIn_SyncParent`'s `fx.puller.opts != (fabricengine.SyncOptions{})` check becomes a field-wise check of `SkipGit` and `SkipPush`.
The four existing tests change: the diverged-weft test makes its divergence conflict on `status.json`, so the replay stops and `ErrPushRejected` still surfaces unwrapped, and asserts the record; the diverged-warp coalesce test asserts the code-side record; the pair test's retry rows follow the records-side replay; the Bolt seed test asserts the seed commit dropped by the replay.

**ImpactSummary:** Every fabric push path changes failure handling; four unexported helpers are deleted with their only callers on this card, and exported signatures stay, so callers outside fabricengine compile unchanged.

**Commit:** 9: fabricengine: a refused push retries by replay, then is recorded and reported
