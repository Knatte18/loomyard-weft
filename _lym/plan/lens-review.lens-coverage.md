```yaml
lens: coverage
outcome: reported
```

Coverage lens: is every decision, acceptance criterion and Intent branch carried by a card and reached by a planned row that would fail if it broke?

1. **MEDIUM — D2 criterion (adopted run resumes at origin's last transition and re-runs that row): no planned row.**
   Card 7's Intent says "Run adoption is unchanged … the adopt tests keep covering it" and names no test. That is a criterion argued without a test.
   Evidence: `_lym/plan/07-transitions-start-pusher.md`, Intent paragraph beginning "Run adoption is unchanged".

2. **MEDIUM — D3 criterion "a batten transition starts the prime's pusher and a loom transition the pair's": no row checks the target.**
   Card 7's rows only assert that `StartPush` is called. The target choice, `runpusher.TargetOf(location, "")` versus `TargetOf(location, slug)`, lives in `internal/battencli#battenCommitStatusDeps` and `internal/loomcli#loomCommitStatusDeps`. No row reads it, so swapping the two targets fails nothing.
   Evidence: card 7's Tests list. `TestNewCommitStatusSeam` and the loom seam tests run on fake deps.

3. **MEDIUM — D5 and D8 "leaves `PushOwed` unset in the run's read" and "with origin unreachable, the run walks its next rows and the dobby still schedules it": no row.**
   Card 1 asserts that no record is written. No row runs `ReadRun` or `runfsm.Judge` after a push made in the new mode, and no row shows a run going on while its push is owed. D8 has no card row at all.
   Evidence: card 1's Intent ("So `FailedPushesOwedBy` returns no record … leaves `PushOwed` unset") is argued, not tested. `internal/pairteardown#TestReadRun` gains rows only in card 9.

4. **MEDIUM — Card 1 branch: the `recordOwedRecordsPush` skip in the new mode has no row.**
   `recordOwedRecordsPush` is called from `recordsSideNotLanded`'s transport lookup and from the records loop's "unpushed past push" deferral at `attempt == len(delays)`. The planned rows reach the transport class through `pushJob.fail` (via `checkOrigin`), the teardown lock through `teardownLockOwedError`, and the deferral through `pushJob.deferral`. Neither of `recordOwedRecordsPush`'s call sites is reached, so that function could still write a record in the new mode and no test would fail.
   Evidence: `internal/fabricengine/pushanchored.go` `recordsSideNotLanded` (transport branch) and the `PushPairAnchored` loop's `attempt == len(delays)` branch; card 1's row list.

5. **MEDIUM — D6 batten criterion "a failed … archive push or hand-over replays nothing and is retried": only the failed takeover read has a row.**
   Card 4's branch where `HandOver` fails and `PushRun` returns `ErrTakeoverPassOwed` with nothing pushed has no row. Card 3's failed `PrimeRecords` archive in `HandOverTips` has no row either.
   Evidence: card 3's rows cover only "the takeover read failing". Card 4's only row is the success row.

6. **MEDIUM — D6 loom criterion "the pair's pusher hands the run over … with `InsidePair` true and exits": the wiring is untested.**
   `TestPusherDeps` asserts only that `Guard`, `Push` and `Notify` are present. Nothing checks that the pair `Push` passes `InsidePair: true`, `OwedUnrecorded` or the slug. With `InsidePair: false`, the pusher would remove the pair from under the run, and no row would fail.
   Evidence: card 6's `TestPusherDeps` paragraph.

7. **MEDIUM — Card 8 dobby branches have no row.**
   These branches in the `DobbyRuns` call go untested:
   - it starts only for a pair that stands here and is not owned elsewhere;
   - a failed start logs one Warn per episode and keeps the run in the tick's views;
   - the prime `Tick` joins the pusher start's error with its others.

   The card says the wiring "adds no branch of its own beyond the call", but the guard condition and the keep-the-view-on-failure path are branches. The D5 criterion "a dobby tick starts a pusher for a prime or pair ahead" is therefore reached only through `TestStart` and `TestRunCommitsAhead`, never through the tick.
   Evidence: `_lym/plan/08-restarts-start-pusher.md`, its last Intent line. There is also an operand note: D5 says "every pair of its runs that is ahead", while card 8 adds "not owned elsewhere", a condition the record does not name.

8. **MEDIUM — Card 6 branch: `resolvePersistentPreRun` skipping `pusher` has no row.**
   If the skip is missing, the verb arms a run and refuses its cwd, the hub's Bolt directory, so no pusher ever pushes. Every tier0 row fakes the spawn, so nothing would fail. The other new exit, "a worktree that does not resolve fails the verb", is unreached too.
   Evidence: card 6's Intent bullets and its single test, `TestPusherDeps`.

9. **LOW — Card 5 `Run` branches: some reads and the lock have no failure row.**
   Planned rows cover "conditions failing", "lock held" and "worktree gone". No row covers an error returned by `Ahead` or `Conditions`, either mid-loop or on the exit re-read, or an error from `Lock` (as opposed to the lock being busy).
   Evidence: card 5's `TestRun` row list.

10. **LOW — Card 2 branch: "a records tip with no `Warp-SHA` trailer counts no code side" has no row in `TestRunCommitsAhead`.**
    Evidence: card 2's Intent, versus its `TestRunCommitsAhead` rows.

11. **LOW — Card 3 branch: `withoutFinalizeEntered` keeping `EndedHandedOver` entries has no row.**
    This needs a handed-over batten run whose local landing entered Finalize. If the entry is dropped, that run's commits are replayed onto origin, against D6's "never replayed".
    Evidence: card 3's Intent bullet on `withoutFinalizeEntered`; `internal/fabricengine/replay.go` calls `withoutFinalizeEntered(l, ended)` in `replayWeft`.

12. **LOW — Card 10 wire branches: "a pair already gone pushes nothing and returns nil" and "`Level` true for a pair already gone" have no row.**
    These carry the resumed-row-after-removal path the card cites, and the producer rows use fakes only.
    Evidence: card 10's Intent bullet on `battenCLI.wire`.

13. **LOW — D5 resume criterion "starts the pusher before the run acts" (loom): the order is not asserted.**
    The `TestResumeVerb` rows count `startPusher` calls but do not assert that the call comes before the driver wake.
    Evidence: card 8's Intent ("before any driver wake") versus its `TestResumeVerb` rows.

I made no further tool calls.
