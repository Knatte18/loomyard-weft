```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM**: card 6, the `PusherSync` sentence: "pushes the Bolt once through `Bolt.Push` with `OwedUnrecorded` and the single transport attempt card 5 gives the pusher's pushes".
- **Summary:** `Bolt.Push` cannot ask for a single attempt.
- **Evidence:** Card 5 puts `SingleAttempt` on `fabriclayout.PairPush`, and only `PushPairAnchored` passes it on. `Bolt.Push(opts SyncOptions)` (internal/fabricengine/bolt.go:121) takes only `SyncOptions`, which has no such field (internal/fabricengine/fabric.go:74-95), and no card adds one.
- **Effect:** each board push still waits out the backoff in `awaitOrigin`, 1, 2 and 4 s (internal/fabricengine/pushclassified.go:710-735). That breaks D2's "no transport backoff of its own" for the pusher. It also makes the new tier1 unreachable-origin row of `TestPusherDeps_BoardPushCommitsTheBoardWrite` wait out the production backoff, which PATTERN-test-speed forbids.
- **Also:** "once" overstates. `PushRecorded` replays a rejected push again over the push backoff (bolt.go:136-140).

**LOW**: card 6, the `TestPusherDeps_BoardPushCommitsTheBoardWrite` row "with `BOARD_SKIP_PUSH` set, a committed but unpushed Bolt reads level".
- **Summary:** the row sets an environment variable for the whole process, and the card never says the test runs serially or names that global.
- **Evidence:** `ApplySkipEnv` reads `os.Getenv("BOARD_SKIP_PUSH")` (internal/boardengine/skipenv.go:13-20), so the row has to set the variable for the whole process.

The other claims hold:
- `--no-ff --no-commit` leaves a staged merge that `concludeMergeSides` commits; a fast-forward is skipped (internal/gitrepo/merge.go:56-100, internal/fabricengine/mergelifecycle.go:41).
- `landUnderLock` holds the parent-side `MergeOptions` (internal/landingshed/finalize.go:521).
- `DobbyRuns` already reads `shedrun.DriverLock(prime, slug)` (internal/battencli/dobbyreads.go:228).
- `PushRun`'s options reach `awaitOrigin` and the rejection retry through `BackoffSleep` (internal/fabricengine/pushclassified.go:193, 313, 712; internal/fabricengine/pushanchored.go:241).
- `commitDirty` holds the board write lock (internal/boardengine/sync.go:57-68).
- A board write spawns a detached sync (internal/boardengine/board.go:86).

Tool calls:
- One Bash call read `gitrepo.MergeStart`'s outcome logic, the takeover retry sleep and the board write's spawn comment. My context showed none of the three: I needed them to check the no-ff staged-outcome claim, whether the catch-up refusal row meets another production sleep, and the claim that a board write leaves its commit to the sync.
