```yaml
lens: fix-affected-set
outcome: reported
```

Task: under the fix-affected-set lens, rerun the search behind each set-premise finding and check that the fix covers every member of the set.

Unchecked findings: none. F1, F2, F3, F5 and F6 were each searched once; F4's premise is a single clearing rule in card 11, and the changed step 1 and `SettleRefused` text covers both of its sites.

Covered sets:
- **F5:** `sheddriver.Deps{` is built in `internal/loomcli/drivewiring.go:236` and `internal/battencli/drive.go:258`. Card 2 now edits both.
- **F2:** `MergeOptions{` is built in `internal/landingshed/finalize.go:521` and `internal/fabriccli/merge_verbs.go:238`. The second is the operator's `lym fabric merge` and makes no landing, so the Finalize site is the only member.
- **F1:** every board spawn goes through `SpawnSync` (card 7), and `starter.start` with `ifAhead` reads the same widened `Ahead`.

Findings:

1. **MEDIUM**, card 6, the new `PusherSync` sentence: "pushes the Bolt once through `Bolt.Push` with `OwedUnrecorded` and the single transport attempt card 5 gives the pusher's pushes".
   - **Summary:** nothing carries that single attempt to a Bolt push, so the new tier1 row's unreachable-origin leg still runs the production backoff.
   - **Evidence:** card 5 puts `SingleAttempt` on `fabriclayout.PairPush`, which only `PushPairAnchored` reads (card 5, line 63). `Bolt.Push` takes only `SyncOptions` (`internal/fabricengine/bolt.go:121`). `SyncOptions` has no such field (`internal/fabricengine/fabric.go:74-95`).
   - **Why it matters:** a Bolt push runs `pushClassified`, then `checkOrigin`, then `awaitOrigin` (`internal/fabricengine/pushclassified.go:123,670`). `awaitOrigin` sleeps `PushBackoffDelays` through `opts.BackoffSleep` or `time.Sleep` (lines 710-733). So `TestPusherDeps_BoardPushCommitsTheBoardWrite`'s "unreachable origin" leg waits out the production 1, 2 and 4 s retries, against PATTERN-test-speed. That is the same defect as F6, and its "about three seconds" estimate does not hold.
   - **Fix:** card 6 (or card 5) has to name a `SyncOptions` field for the single attempt that `PusherSync` sets, or the test has to pass a `BackoffSleep`.

2. **LOW**, card 11, the new opening paragraph: "Every owner read of steps 1 to 4 reads that pulled state".
   - **Summary:** the re-record paragraph that follows it was not updated to say that its `done` board-entry read also reads the pulled Bolt.
   - **Evidence:** the re-record skips runs "whose board entry is not `done`" (card 11, re-record paragraph). The new text says the pull comes "before the re-record", but it only says that owner reads use the pulled state. A reader could still take the board-entry check as a read of the local board.

Tool calls:
- Grepped `sheddriver.Deps{` across `internal`: the F5 set of driver constructions was not in context.
- Grepped `MergeOptions{` across `internal`: the F2 set of parent-side merge sites was not in context.
- Grepped the plan for "unreachable": the F6 set of test rows over an unreachable origin was not in context.
- Grepped the plan for `HasUnpushed`, the board pusher and `SpawnSync`: the F1 set of board-ahead readers was not in context.
- Read `pushclassified.go:655-675` and `pushClassified`'s head: needed to confirm that the Bolt push path reaches the `awaitOrigin` backoff, which was not in context.
