```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches its branch and would fail if the code were wrong, and whether any new test repeats an existing one. Nine findings:

1. **MEDIUM — Card 5, `LineFits` measuring the marked form.** An existing row breaks, and no new row pins the change.
   - The row `Post_RefusesEmptyAndOversizedBody` in `internal/creelengine_test#TestStore_Mail` (`internal/creelengine/verbs_test.go:145-158`) works out its overhead from the unmarked `MessageLine`. It then asserts that the body `budget-overhead` posts.
   - Once `LineFits` measures the longer marked line, that body is refused. The row then fails on card 5's own gate, and the card's Intent never says to rewrite it.
   - No planned row posts a body that fits the plain line but not the marked one. A `LineFits` that kept measuring the plain form would pass every planned row.
   - Fix: card 5 rewrites that row to take the overhead from the marked form. It also adds a row where a body that fits only the plain line is refused.

2. **MEDIUM — Card 9, sweep rows with a chaser running.** The rows cannot drive the sweep with the existing helpers.
   - `env.sweepStep` and `env.sweepLast` pick waits through `isSweepWait`, which is `d &gt; 8*time.Second` (`internal/creelengine/deliver_test.go:25`).
   - Card 9's rows have a read, marked message, so card 7's chaser runs beside the sweep. The chaser's one-minute waits also match `isSweepWait`.
   - So `sweepStep` can advance the clock by a chase wait instead of the sweep's 30–50 s or 10 s confirm wait. That breaks the two-miss timing and charges the chase clock along the way.
   - Fix: card 9 (or card 7) edits `internal/creelengine_test#isSweepWait` to `8s &lt; d ≤ 50s`, beside card 7's `isChaseWait`.

3. **MEDIUM — Card 7, the restart and crash rows in `TestScheduler_Chases` need a journal-writing helper the card does not list.**
   - Three rows need it: "a chase `posted` record with no `chased` record", "a consumed clock past the next step before the restart", and "a store reopened after a pause over the same journal".
   - Each needs hand-written journal lines or a closed and reopened store under the external package's `env`.
   - `env` has neither: `writeJournal` is a method of `server_test.go`'s `rig` (`internal/creelengine/server_test.go:240`). `env.start` also registers `scheduler.Stop` only at cleanup, so nothing stops the first scheduler and store before the reopen.
   - The crash row cannot come from any natural sequence, so it cannot reach `repairChases` without such a helper.
   - Fix: the card creates an env journal-write helper and a stop-and-reopen helper as `plan:` handles.

4. **MEDIUM — Card 7, `isChaseWait` rests on an unproven claim.**
   - The card says "every chase wait is a minute" because the rows advance the clock by whole minutes from the read.
   - The chaser's wait is the lesser of the time to the next step and 60 s. A probe off the minute grid gives a wait under 60 s, which `isChaseWait` (over 50 s) misclassifies. That probe can come from a wake signalled by `Reading.Ack`, `markNoticeRead` or a reply after the clock started.
   - The rows "a reply after the first chase stops the second" and "pause / dormant / hold stops the clock" are exactly those mid-interval wakes.
   - Fix: state that a wake recomputes the wait on the minute grid, or classify chase waits by caller rather than by length.

5. **MEDIUM — Card 11, the `repliesCommand` verb has no row.**
   - Only `TestComputeReplies` is planned.
   - Nothing reaches the verb's prime-only refusal naming `lym report replies`, its `Log` read through `NewProgramClient(..., "REPORT")`, or its `roles` envelope.
   - The existing `TestReportVerbs_RefuseOutsideThePrimeThenReportTheSeededRun` drives only `convergence` and `timing` (`internal/reportcli/convergence_integration_test.go:67`, `:202`).
   - Fix: a row in that test, or in `TestReportVerbs_RefuseBadFlags`, runs `replies` from a pair (refused) and from the prime (an ok envelope with `roles`).

6. **LOW — Card 13 adds a repeat row.** The new "requiring `peer-messaging` is refused at start as unknown" row repeats the existing unknown-requirement row (`"telepathy"`, `internal/shuttleengine/requires_test.go:86`). The same branch is reached with another string. Fix: change that row's input to `"peer-messaging"` instead of adding a row.

7. **LOW — Card 1, `orchroute#TestAsker` repeats rows.** Its pair rows (`m` with `b` live, `m` with `b` not live, `p`) repeat `internal/orchcli#TestParentOrchStrand`'s rows over the same walk (`internal/orchcli/notify_integration_test.go:72-113`), which delegates to `FromPair` after this card. Only the beam and prime rows, which go through `From`, and the not-recorded row are new behavior. Fix: keep only those.

8. **LOW — Card 6 plans no row for `read --envelope`.** The new `expect_reply` message field is untested. Only the text header (`TestWriteMessagesText`) and the `log` fields get rows, so a `messageFields` that drops the field passes.

9. **LOW — Card 2's refusal rows cannot assert "no posted event was journaled".** The first two refusals run "with no daemon started", but checking the journal through `creel log` starts the daemon. The row needs a stated way to read the journal: start the rig daemon after the refusal, or read the journal directory.

Calls I made:
- Read the enclosing functions of `run_test.go:1023` and `run_inject_test.go:91`, the line-budget tests, `rig.writeJournal`'s package, and the report verb tests: my context lacked the test-function names, the budget rows and the report test coverage.
- Read `verbs_test.go:140-165` and `:1105-1130`: my context lacked the bodies of the two budget rows that card 5's `LineFits` change touches.
