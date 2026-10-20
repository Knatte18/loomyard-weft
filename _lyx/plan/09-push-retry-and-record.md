# Card 9 — push-retry-and-record

**Create:**
- `plan:internal/fabricengine#ErrBranchGone` -> `var ErrBranchGone error`
- `plan:internal/fabricengine#goneBackoffSleep` -> `var goneBackoffSleep func(time.Duration)`
- `plan:internal/fabricengine#pushClassified` -> `func pushClassified(rec *Mutations, hubPath, side, path string, retryRecords func() (deferred bool, err error)) error`
- `plan:internal/fabricengine#branchGoneOnOrigin` -> `func branchGoneOnOrigin(path string) (gone bool, err error)`
- `plan:internal/fabricengine#goneEntryEnded` -> `func goneEntryEnded(hubPath, branch string) (bool, error)`
- `plan:internal/fabricengine#retryWeftPush` -> `func retryWeftPush(rec *Mutations, hubPath, weftPath string) (deferred bool, err error)`
- `plan:internal/fabricengine#retryBoltPush` -> `func retryBoltPush(rec *Mutations, boardDir string) (deferred bool, err error)`
- `plan:internal/fabricengine#PushFailureRewound` -> `const PushFailureRewound PushFailureClass = "rewound"`
- `plan:internal/fabricengine#SetGoneBackoffSleepForTest` -> `func SetGoneBackoffSleepForTest(sleep func(time.Duration)) (restore func())`
- `plan:internal/fabricengine_test#TestPushFailureClasses` -> `func TestPushFailureClasses(t *testing.T)`

**Edit:**
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
- `plan:internal/fabricengine#FailedPushWayForward`
- `internal/orchcli#writeHubRules`
- `internal/orchcli#TestWriteHubRules`
- `internal/fabriccli#reconcileFailureWayForward`

**Delete:**
- `internal/fabricengine#pushRebaseFreeLogged`
- `internal/fabricengine#unpushedOutcome`
- `internal/fabricengine#pushWeftAt`
- `internal/fabricengine#Bolt.dropSeedCommitsAfterConflict`

**Uses:**
- `plan:internal/fabricengine#RecordPushFailure`
- `plan:internal/fabricengine#ClearPushFailure`
- `plan:internal/fabricengine#replayWeft`
- `plan:internal/fabricengine#replayBolt`
- `plan:internal/fabricengine#readEndedSet`
- `plan:internal/gitrepo#ErrPushAuthRefused`
- `plan:internal/fabricengine#trackingReflog`
- `plan:internal/fabricengine#classifyRewrite`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D10: every push fabricengine runs classifies a failure, recovers where it safely can, and otherwise writes the failed-push record and its one notice; D5's replay replaces `gitrepo`'s pull-rebase on every fabric push path.

`pushClassified` is the one classifier the detached coalesced push (`coalescePushSides`), `PushAnchored`, `PushPairAnchored` and Bolt's pushes (`Bolt.PushRecorded`, reached by `Bolt.Push` and Bolt's syncs) run per side:
- Before pushing a weft or a task branch with an upstream it runs `branchGoneOnOrigin`, one `git ls-remote origin` of that branch; Bolt skips it, since Bolt's branch is never deleted.
  A gone branch is not pushed, so a parked push never recreates a branch landed or deleted elsewhere: `goneEntryEnded` fetches origin's Bolt branch (one fetch, the gone path's only Bolt refresh) and reads the ended set by D3 step 2's rule; an ended entry writes no record and is left to D6's teardown; otherwise the judgment is the push's own transport failure, retried by the backoff (`goneBackoffSleep`, the delays `gitrepo.PushWithRetry` uses, injectable) re-running the ls-remote, the Bolt fetch and the judgment, and only a branch still gone with its entry neither done nor removed at the end writes a `gone` record.
  A transport failure of either call is the push's own transport failure.
- A task branch's push is not run when `classifyRewrite` over the origin head that ls-remote returned and `trackingReflog` reads the branch as **rewound**: origin's head is then an ancestor of the local tip, so a plain push would fast-forward origin and restore the commits another machine's leased rewind dropped (D4's bound, and the criterion that its push does not restore them).
  It writes a `rewound` record, which mails, and `FailedPushWayForward` gains the class, naming `ResetPairCode`'s way forward; the branch's next push after the rewind is settled lands and removes the record.
  Bound: only a task branch read as rewound is held back; a weft, Bolt and the prime's code branch are never judged so.
- A records-side rejection (`gitrepo.ErrPushRejected`) calls the side's retry once: `retryWeftPush` fetches the weft branch and origin's Bolt branch, try-acquires the weft write lock while holding the push lock, reads the ended set from the fetched Bolt tip and calls `replayWeft`, then pushes again; `retryBoltPush`, holding `board.push.lock`, try-acquires `board.lock`, then takes `.gitrepo-push.lock`, fetches, reads the ended set and calls `replayBolt`, then pushes again.
  A held write lock or `ErrReplayDirty` is a deferral: the holder writes no pending flag of its side and ends its loop, leaving the commits to the next commit's push, and writes a `deferred` record (no mail), which the next landed push removes.
  A replay that stops (`ErrReplayConflict`) or a second rejection is a failed push of class `not_fast_forward`, and the returned error still wraps `gitrepo.ErrPushRejected`, so `IsPushRejected` callers keep working.
- A code-side rejection is a failed push at once; `pushRebaseFreeRetryingOnce` keeps only its stale-tracking-ref retry, now for the code side alone.
- `ErrPushRetriesExhausted` and a `gitexec.IsTransportFailure` error are the existing backoff's failure, class `transport`; `gitrepo.ErrPushAuthRefused` is class `auth` at once, with no retry.
- A push that lands calls `ClearPushFailure` for its repo and branch.
`coalescePushSides` keeps returning nil for a failed push, as the detached child always did, but the failure is now recorded rather than only warned; `PushAnchored` and `PushPairAnchored` return the error as before and record it too.
`Bolt.PushRecorded` pushes with `PushRebaseFree` instead of the pull-rebasing `pushWeftAt`, and its seed-commit recovery becomes `retryBoltPush`'s replay, so `dropSeedCommitsAfterConflict`, `pushWeftAt`, `pushRebaseFreeLogged` and `unpushedOutcome` are deleted with their callers here, and `PushWarpAt`'s doc comment and `spawn.go`'s file header, which describe it as the sibling of `pushWeftAt` with matching gating, are reworded to stand alone.
With Bolt's push no longer rebasing, `gitrepo.ErrPullRebaseFailed` never reaches `orchcli`'s `writeHubRules` from `bolt.Push()`: its `diverged` branch and the way forward naming `git pull --rebase` are replaced by a branch on the failed-push record's class that names that class's way forward (`FailedPushWayForward`), and `TestWriteHubRules`' fake push returns a rejection carrying a recorded class instead of `ErrPullRebaseFailed`.
`fabriccli`'s `reconcileFailureWayForward` drops its `git pull --rebase` clause for a rejected board push and names the failed-push record `lyx fabric status` prints, with `lyx board sync` once its way forward is followed.
The hub is the parent of the pushed worktree, so the detached child needs no new flag.
`doc.go` replaces the push-retry and seed-drop paragraphs with this classification, and `PATTERN.md` gains the `PATTERN-push-failure-visible` entry line in its "Fabric and git" section, worded exactly as the overview's PATTERN changes bullet with `(test)`, within the entry line cap.
Bound: the retry replays only records branches, once per push, never waits on the write lock; a deferral costs at most one rejected push per commit; the notice is one posted mail per branch per episode.

`TestPushFailureClasses` (integration, `hubforge` hub, a second clone of each bare as the other machine, `boardkit.InstallReplay` called first, a fake notifier installed through `InstallPushNotifier` counting only its own hub's calls, `goneBackoffSleep` replaced through `SetGoneBackoffSleepForTest`; run without `t.Parallel`, naming the process-global notifier) is the enforcing test.
Its transport input is the pushed worktree's `origin` URL set to `http://127.0.0.1:1/x`, which git refuses with `connection refused`, a phrase `gitexec.IsTransportFailure` matches; the gone-path rows restore the URL inside the injected sleep where a row needs a later attempt to answer.
Rows: a detached weft push rejected as not fast-forward is fetched, replayed and pushed; with the weft write lock held it defers, writes a `deferred` record with no mail and sets no pending flag, and the next commit's push lands it and removes the record; a detached weft push whose replay meets a both-sides `status.json` change writes a `not_fast_forward` record and mails; a records retry whose second push is rejected again (the other machine pushes once more inside the retry, through a pre-receive hook on the records bare that admits one push) writes a `not_fast_forward` record; two concurrent failures of one branch under a 403 hook write one record whose count is two and post one mail, and a landed push removes it; a push to the refused URL writes a `transport` record after the backoff; a parked push of a branch gone on origin does not recreate it, and one of an entry ended on origin writes no record; a gone branch whose Bolt fetch meets the refused URL is retried as a transport failure and records nothing until the backoff ends; a parked weft push of a run done on origin since this machine's last Bolt fetch writes no record; a weft push whose branch origin deleted before its Bolt shows done backs off, and the done mark arriving during the injected sleep ends it with no record; a prime-weft push rejected right after this machine's Finalize pushed `done` keeps the landing's commits; a Bolt push rejected over an unrelated origin entry is replayed and lands; a Bolt push rejected while `board.lock` is held defers with a `deferred` record; a task branch rewound on origin by a leased update, pushed from this machine with the dropped commits still local, is not pushed, origin's head stays where the rewind put it and a `rewound` record names `ResetPairCode`'s way forward.
The code-side rejection's record is asserted in the changed coalesce test below, not again here.
The four existing tests change: the diverged-weft test makes its divergence conflict on `status.json`, so the replay stops and `ErrPushRejected` still surfaces unwrapped, and asserts the record; the diverged-warp coalesce test asserts the code-side record; the pair test's retry rows follow the records-side replay; the Bolt seed test asserts the seed commit dropped by the replay.

**ImpactSummary:** Every fabric push path changes failure handling; four unexported helpers are deleted with their only callers on this card, and exported signatures stay, so callers outside fabricengine compile unchanged.

**Commit:** 9: fabricengine: a refused push retries by replay, then is recorded and reported
