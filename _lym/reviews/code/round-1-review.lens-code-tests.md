```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none. I made no tool calls and judged from the inherited context.

**Behaviour → pinning test**

| Behaviour | Pinned by |
|---|---|
| VSTest counter keeps each `Failed <name> [dur]` block: section headings dropped, block closed by a blank or summary line, a `Passed` line before the block ignored | `TestCounters`, row "each VSTest failed test is named with its block's lines…" (`counters_test.go`) |
| VSTest block capped at `failureLines` | none |
| VSTest block closed by a following `Passed`/`Skipped`/`Failed` result line with no blank between | none. The row's second `Failed` follows a blank line, so the result-line close path is never taken. |
| VSTest `Failures()` returns a copy under the lock | none, and a copy test would be low value |
| Run record `failures` carries VSTest tests (`runlog.go` comment change) | none seen in the diff. The generic record path may cover it. |
| Report renders a package-less failure as the test name alone (`failurereport.go` TrimSpace change) | `TestRenderFailureReport`, row "a failed test with no package…" |
| `TierRunFailures`: label per failure, failure outside any test, run named by outcome when it lists none, cut line, build/vet/lint/no-report give false | `TestTierRunFailures` |
| `TierRunFailures` on a header with no runs, on a run's cause line ending in a colon, and on a blank line ending the report | none. `reportFailureLine` accepts any `  x:` line except `its output:`. |
| Rerun of Go tests alone via `FailedTestsCall` | `TestVerifyGate_RerunsATestFailureOnce`, row 1 |
| Rerun of the whole call when no Go test is named | same test, row 2 |
| A repeated failure is judged on the rerun's log | same test, row 3 |
| Build failure not rerun | same test, row 4 |
| Timeout not rerun | same test, row 5 |
| No rerun when `onFlakyPass` is nil | same test, row 6 |
| Runner crash not rerun | none |
| Shell start failure (`ExitCode < 0`) not rerun | none |
| `Log == ""` or unreadable log means no rerun | none |
| `FailedTestsCall` error falls back to the whole command | none |
| Rerun status `Skipped` counts as flaky | none |
| `verify` error on the rerun is returned wrapped | none |
| Rerun fails differently, such as a timeout or crash on the second run, and is judged as that failure | none. Row 3 only repeats a plain failure. |
| Flaky pass: no fixer, two verify calls, section in the landing message | `TestFinalize_FlakyGateFailureLands` |
| Flaky section in the GitHub merge title and body | none. The test asserts only `merger.calls[0].opts.Message`. |
| `NewFinalize` wires `gate.onFlakyPass` | none. The test assigns `fx.fz.gate.onFlakyPass = fx.fz.noteFlakyPass` itself, so deleting the wiring in `NewFinalize` leaves it green. |
| `flakySection` dedupes failures across passes and returns empty when there were none | dedup: none. Empty: only indirectly, through unchanged older landing tests. |
| Parent-Align's gate runs no rerun | none at the producer level. Only the nil-sink row at gate level. |
| The second `landingTexts` call that refreshes `mergeOpts.Message` in the retry path | none |
| Spec and doc prose changes | exempt |

**Findings**

1. MEDIUM, design. `internal/landingshed/finalize_gate_test.go` (the `fx.fz.gate.onFlakyPass = fx.fz.noteFlakyPass` line in `TestFinalize_FlakyGateFailureLands`). The test installs the sink itself, so the `NewFinalize` wiring (`finalize.go`, `fz.gate.onFlakyPass = fz.noteFlakyPass`) has no test. Removing it would disable the whole feature with the suite green. Fix: have the fixture build the gate through `NewFinalize`'s wiring and drop the manual assignment.
2. MEDIUM, design. `internal/landingshed/verifygate_test.go`, `TestVerifyGate_RerunsATestFailureOnce`. The commit message states that a runner crash is not rerun, and `rerunTestFailure` also skips `ExitCode < 0` and `Log == ""`. None of these has a row. Removing the `RunnerCrash` guard would rerun a crash and then mask it as flaky. Fix: add rows for `RunnerCrash: true`, `ExitCode: -1` and an empty `Log`, each expecting one verify call and a reason.
3. LOW, design. Same test. The branches "`FailedTestsCall` errors → whole command", "rerun status `Skipped` → flaky" and "rerun fails differently (timeout or crash on the second run) → judged as that failure" are untested. Fix: add a row where the rerun result is `TimedOut` and expect a timeout reason naming the second log, and a row where the rerun result is `Skipped` and expect a flaky pass. The `FailedTestsCall` error branch is hard to reach and can stay uncovered.
4. LOW, design. `TestFinalize_FlakyGateFailureLands`. The flaky section in the GitHub merge title and body, which comes from the first `landingTexts` return values, is not asserted. Nor is dedup across two flaky passes in `flakySection`. Fix: assert the body passed to the GitHub merge path contains `flakyHeading`. Add a `flakySection` case with two passes sharing a failure, expecting one bullet.
5. LOW, design. `TestTierRunFailures`. Missing rows: a report header with no runs (expect `ok` false), and a cause line ending in a colon under a run. The second one would be wrongly taken as a failure and mark the run named. Fix: add both rows. If the cause-line case gives a wrong result, that is a `design` bug for the code lens.
6. LOW, design. `TestCounters` VSTest row. The `failureLines` cap and the close on a following result line with no blank between are not exercised. Fix: add a block with more than eight lines, and a `Failed` block directly followed by a `Passed` line, expecting no later line to attach to the failed test.

**Test economy.** I found no duplicated coverage. The redundancy report has no coverage map for these packages, so it judges none of the tests.

**Tool calls:** none.
