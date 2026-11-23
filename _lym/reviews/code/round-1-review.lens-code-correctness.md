```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness review of the diff through the code-correctness lens. Per function or branch: the case I ran, and whether it came out correct.

## Cases run

- `runpusher.Run` / `pushUntilLevel` / `workRemains`:
  - A commit made after the final ahead read but before the lock release is picked up by the post-release re-read. Correct.
  - A pause stops the pass without an error. Correct.
  - A guard stop returns its error before pushing. Correct.
  - A refused push or a hand-over returns the error. Correct.
  - A worktree that is gone returns nil. Correct.
- `PushUntilLanded`: the backoff runs 1, 2, 4 … 60 s and the notice fires once at 60 s. Correct.
- `runpusher.Start`:
  - Skip env, lock, refused-push record, pause and hand-over mark each stop the spawn. Correct.
  - Two racing starts may both spawn and the loser exits at the lock. Correct.
- `RunCommitsAhead`, `localBranch.ahead/lacks/heldOnOrigin`:
  - No commits, no upstream, a pruned ref and a tracked ref each read as intended. Correct.
  - A pair whose records tip carries no `Warp-SHA` counts no code side. Correct.
- `RefusedPushStands` ignores owed classes and matches branches by repo side. Correct.
- `checkOrigin` with a pruned ref pushes and skips the rewind check. Correct.
- `checkOrigin` with an ended entry deletes the tracking ref and keeps the upstream. Correct.
- `PairPush.OwedUnrecorded` records no owed-class failure and records the other classes. Correct.
- `primePushTakeovers` and `PushRun` (prime loop): a pending takeover is handed over, the mark is written, and the re-read lists the run as handed over, so the loop terminates. A failed read, archive or hand-over wraps `ErrTakeoverPassOwed`, which reads as owed. Correct.
- `statuscommit.New`, the batten seam and `commitRunRecords` commit then start the pusher, with no push and no fetch. Correct.
- The loom `commit-records` verb and `pusherDeps` fill their seams as intended. Correct.
- `loomCLI.resumeFromRunState` starts the pusher after the refusals and before any wake. Correct.
- `battenPreRun` starts the pusher after the owner pre-flight and before the reopen. Correct.
- `runread` `localHoldsTip`: the case where origin's missing tip is absent from the local repo is not demonstrably correct. See F1.
- `worktreeTeardownProducer.Call`: a Level read that never turns true loops without end. See F2.

## Findings

**F1 — MEDIUM — design — `internal/pairteardown/runread.go` (the `localHoldsTip` seam in `hubRunReader`, its call in `read`)**
- `localHoldsTip` calls `fabricengine.HeadContains(WorktreePath(prime, slug), tip)` directly.
- The tip is origin's last-transition `Warp-SHA`, which this machine's code branch may never have held.
- In `RunCommitsAhead`, `holdsCommit` (`cat-file -e`) runs before `HeadContains`, which suggests `HeadContains` errors on an absent object.
- If it does, a missing tip the local pair does not hold turns a normal read into the failure "whether the local code branch holds the code tip".
- The run is then unreadable, and the dobby tick leaves it out.
- The old path set `CodeTipMissing` and let `CodeTipGrace` and the code-tip settle handle it.
- Evidence: `runread.go` hunk at about line 249; `fabricengine/handover.go:390` (`holdsCommit`); `warpforward.go:59`.
- I did not read `gitrepo.HeadContains`, so this rests on that inference.
- Fix: call `holdsCommit` first, or the exported equivalent, and treat an absent object as not held.

**F2 — MEDIUM — design — `internal/battenshed/teardown.go` (the `Call` loop) with `internal/battencli/wire.go` (the `Teardown.Push` / `Teardown.Level` seams)**
- With `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set, `PushPairAnchored` returns at once and pushes nothing (`pushanchored.go:184`).
- `PushRun` then returns nil, and `TipsOnOrigin` reads false for a branch with no upstream or a stale ref.
- `Call` loops with no sleep, taking and releasing the prime lock and re-pushing, and never reaches `Remove`.
- The other failure mode is the same: any `Level` that stays false after a successful push spins the same way.
- Fix: bound the loop, for example by treating pushes-off as level in `Level`, or by returning Stuck when `Level` is false after a push that landed twice in a row.

## Not found
Nothing wrong in the rest of the cases listed above. Cases I could not verify:
- The `TakeoversError` pass-through inside `pushClassified`.
- `moveTrackingRef(…, "", false)` as a delete.
- Whether `HandOver` with `RunPush{PrimeRecords: true}` (zero `LockWait` and `TeardownWait`) takes any lock with a wait.

## Tool calls
1. `grep HeadContains` / `holdsCommit` in `fabricengine`: to see how `HeadContains` and `holdsCommit` guard an absent object. My context lacked their definitions.
2. `grep SkipPush` in `pushanchored.go` and the wait fields in `pairteardown/handover.go`: to check the pushes-off behaviour and the `HandOver` wait fields. My context lacked them.
