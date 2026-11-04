```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour: pinning test**

- Crash detection in `runnerCrash` (128 plus a crash signal, no failing test): pinned by `TestRetryTier` in `internal/gatecli/tierrun_test.go`, rows `CrashThenPassEndsPassed`, `CrashTwiceCarriesTheSignal`, `CrashThenFailingTestCarriesNoCrash`, `SignalExitWithAFailingTestIsNotRerun` and `TerminatedRunIsNoCrash`.
- The one rerun cap and both runs' records (`recordFirst`, `Rerun`, `RunnerCrash` on the first record): pinned by `TestRetryTier`, which checks `recorded` and the rerun mark.
- The rerun log opening with the announcement, and the crash announcement text: pinned by `TestRetryTier`.
- Context cancelled during the delay for a crash: none. Only the shortage case (`CancelDuringTheDelayReturnsTheFirstResult`) is pinned.
- `failedRunLabel` crash wording: pinned by `TestFailedRunLabel`, row `CrashTwice`.
- `FindRunnerCrash` (CRLF, cut at `;`): pinned by `TestWatchdogLine`, rows `RunnerCrash_*`.
- `RunnerCrashLine` content: only through that round-trip.
- `exitCodeOf` returning 128 plus the signal for a direct child killed by a crash signal, and 1 for other signals: none.
- `runTest` crash accounting in `run.go` (`crashed`, `judged`, closing `RunnerCrashLine` only when no judged failure): none. The diff changes no `run_test` file, so nothing covers any of these cases:
  - crash plus a failing test in another tier;
  - crash plus a lint finding;
  - crash plus a blocked build.
- `report` setting `WaitStart = Start` on a rerun record, and `Observer.Ended` firing once per rerun tier: none.
- `verifytree.runnerCrashDetail` and `Verify` setting `RunnerCrash` with the timeout line taking precedence: none. The diff has no `verifytree` test.
- `detachfixer.endedOnRunnerCrash` (no fixer spawned on a crash line): none.
- `verifyGate.check` returning `crashed`, and `verdict` wording: pinned by `TestVerifyGate_CheckMapsVerifyResult`, row `runner crash is a halt reason ...`.
- `recordFailure` skipping a crash: only by the Finalize test's `ReadLandingFailure` assertion. It is not pinned in the verifygate unit test.
- Finalize halting with no fixer and no landing failure record: pinned by `TestFinalizeVerify_RunnerCrashHaltsWithoutFixer`.
- Parent-Align crash halt (`stuckVerdict` on `crashed`): none.
- Fixer-turn crash (`fixerEndCrash`, `haltReason` returning `o.crash`): none. The merge fixer's turn gate is uncovered.
- `checkHead` / `headFailureReason` crash wording for the pr-review gate: none.
- Loom `verifyFailureFindings` crash text: pinned by `TestVerifyFailureFindings`.
- Loom `NewVerifyGate` returning `Terminal: res.RunnerCrash` (the row spawns no fixer): none. Only the findings text is asserted.

**Findings**

1. MEDIUM, design. Location: `internal/gatecli/run.go:150-206`. The closing-line rule is untested: `RunnerCrashLine` is emitted only when every failed run crashed and no other failed run, lint or blocked build judged. Evidence: no gatecli run test in the diff. Every site's crash detection reads this line, so a regression in this rule silently sends a crash to a fixer or hides a real failure.
2. MEDIUM, design. Location: `internal/verifytree/verify.go:419-423` and `:507-518`. `Verify` setting `RunnerCrash` and `Detail` from the log, and TimedOut taking precedence, has no test. Evidence: no verifytree test change. Every landing and loom site depends on this reading.
3. MEDIUM, design. Location: `internal/loomshed/verifygate.go:141`. `Terminal: res.RunnerCrash` has no test, so the loom row's "no fixer" guarantee is unpinned. Evidence: `loomshed/verifygate_test.go` only asserts `verifyFailureFindings` text.
4. MEDIUM, design. Location: `internal/landingshed/parentalign.go:198-200` and `internal/landingshed/fixerrun.go:291-294`. The Parent-Align crash halt and the merge fixer's turn-gate crash end (`fixerEndCrash` and its `haltReason`) have no test. Evidence: the only landing crash tests are Finalize's and the `check` mapping.
5. MEDIUM, design. Location: `internal/gatecli/detachfixer.go:125-133`. `endedOnRunnerCrash`, which stops the card fixer being handed a crash, has no test. Evidence: no detachfixer test change.
6. LOW, design. Location: `internal/gatecli/run.go:675-690`. `exitCodeOf`'s new signal branch (128 plus the signal for crash signals, 1 for others) has no test. Evidence: no test change for `exitCodeOf`.
7. LOW, design. Location: `internal/landingshed/verifygate.go:308-313`. The `headFailureReason` crash wording for the pr-review gate has no test. The wording tells the agent to end its turn again, which reruns the gate.
8. LOW, design. Location: `internal/gatecli/run.go:553-557` and `:494-505`. The rerun record's `WaitStart` handling and the single `Observer.Ended` for a rerun tier have no test. A crash with the context cancelled during the delay is also untested.

**Tool calls:** none made. The manifest already held every test diff.
