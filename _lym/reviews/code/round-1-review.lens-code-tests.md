```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none:

1. **Gated, files present, non-awaited shell, inside the bound: not Done and no hold.**
   - Pinned by `TestPollEventsTick_OutstandingBackgroundWork`, `wantDonePastBound` rows (`waiting_test.go:281-283`).
   - Each row asserts "still waiting" on the first tick and on the tick at `bound - 1s`.
   - Breaking the gated branch back to Done-at-once fails them.
2. **Gated, files present, shell past the bound: Done, `countedTasks` set, `waitingTasks` cleared.**
   - Pinned by the same rows (`waiting_test.go:300`).
   - `TestWait_RecordsEndedShells` (gated row, `jump: 6m`, `wantOutstanding: 12m`) pins the end-to-end Done with the shell recorded.
3. **Staggered two shells: Done only once the later one passes the bound.**
   - Pinned by the new staggered gated row (`waiting_test.go:283`).
   - It covers `unawaitedShellsPastBound` checking every shell.
4. **Gated, file missing, shell: held at once.**
   - Pinned by the existing row at `waiting_test.go:284`, which is unchanged.
   - It still covers the reordered branch in `pollEventsTick`.
5. **Gated with a fork or an awaited shell: keeps waiting.**
   - Pinned by the existing rows at `waiting_test.go:285-286`.
   - The new guard `!filesExist || !unawaitedShellsPastBound` has a row on each side of it.
6. **Ungated expiry refactored into `unawaitedShellsPastBound`: unchanged.**
   - Pinned by the existing ungated `wantHeldPastBound` rows, including the staggered one at `waiting_test.go:240`.
7. **Nudge sent once per shell across two waiting turn ends, with the exact text.**
   - Pinned by `TestWait_GatedShellNudgeNamesEachShellOnce`.
   - It asserts exactly one `SendText` call with the exact text, gate called once, and no `EndedShells`.
   - The second waiting turn end does not re-send. This checks `nudgedShells` and also the `shellNudge` clear.
8. **Nudge not queued when a turn start is already read past the waiting turn end (`!turnStartAfter` guard, `wait.go:918`).**
   - Pinned by none.
   - The grep for `turnStartAfter` in the tests found no test that exercises it, and the nudge test has no turn-start signal.
   - No test would fail if the guard were removed.
9. **Pause (`run.covered`) drops the nudge unsent (`sendShellNudge`).**
   - Pinned by none.
   - The grep for "dropped under a pause" in the tests found nothing.
10. **Failed send is logged and not retried (`sendShellNudge`).**
    - Pinned by none.
    - The grep for "not sent" in the tests found nothing.
11. **Nudge text contains the delimited label.**
    - Pinned only through the test's own `shellNudgeText` and `delimitedAgentText`, which the test calls to build its expected value.
    - Reworded text would not fail the test. That is acceptable, since it is static prose.
12. **Nudge not queued for a gated run with a file missing (held branch).**
    - Pinned by none.
    - Row 4 does not check `SendText`.
    - The structural risk is low, because the nudge is queued only inside the files-present branch.
13. **Doc, config and template text.** These are static prose, so they are exempt.

**Findings**

- **MEDIUM, design (test economy):** the `turnStartAfter` guard that skips queueing the nudge has no test. Location: `internal/shuttleengine/wait.go:918`. Evidence: the nudge test (`waiting_test.go:484-535`) has no turn-start signal, and the grep for `turnStartAfter` in the tests matched nothing. Fix: add a row or assertion that a turn start read in the same batch past the waiting turn end sends no `SendText`.
- **MEDIUM, design (test economy):** the pause drop (`run.covered`) and the failed-send path of `sendShellNudge` are both uncovered. Location: `internal/shuttleengine/wait.go:1192-1199`. Evidence: the greps for "dropped under a pause" and "not sent" found no match in the tests. Fix: add a pause-covered case asserting no `SendText` and no panic, and a failing-send case asserting the run still reaches Done at the bound. Both can be small variants of `TestWait_GatedShellNudgeNamesEachShellOnce`.
- **LOW, design:** no test shows that the nudge is not sent for a gated run with an output file missing, or that a fork or awaited shell outstanding queues none. Location: `internal/shuttleengine/waiting_test.go:284-286`. Fix: assert an empty `SendTextCalls` in those rows. Skip it if that needs a new harness.

**Tool calls**
1. `grep` over `internal/shuttleengine/*_test.go` for nudge, drop, send-failure and `turnStartAfter` coverage, and for the `staggered` rows. My context held the diff but not the whole test file.
