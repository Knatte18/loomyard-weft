```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or "none".

**Per behaviour**
1. **A tier child ends within the bounded delay while an orphan holds its pipe (`WaitDelay` on the tier's command).**
   - Pinned by the new subtest in `TestGateTest_Scenario`, "a tier whose shell exits while an orphan holds its output…", at `test_integration_test.go:1550`.
   - With `cmd.WaitDelay` unset, the sleeper keeps the pipe open and `g.wait` never returns, so the subtest fails or hangs.
2. **The kill of the shell's leftover process group after the shell exits.**
   - Pinned by the same subtest, through `eventually(... processGone(record.Grandchild))` at line 1558.
   - Without `killLeftovers`, the sleeper lives on and that assertion fails.
3. **The tier records the shell's exit code and fails.**
   - Pinned by the same subtest: exit 134, a run record with `ExitCode` 134, outcome `failed`, `TimedOut` false (lines 1555 and 1559), and the slots free via `requireFree`.
4. **`exec.ErrWaitDelay` after a zero exit maps to `nil`.** This is the branch `errors.Is(waitErr, exec.ErrWaitDelay) && cmd.ProcessState != nil` in `child.go:57`.
   - Pinned by none.
   - The only scenario uses a failing exit (134), where `Wait` returns the `*exec.ExitError`, so the `ErrWaitDelay` branch never runs.
   - If the branch were removed, or mapped the delay to a failure, every test would still pass.
   - Whether the delay alone, with a zero exit, still yields a passed tier with exit 0 is unchecked.
5. **The build step passes `pipeWaitDelay` to `runTiedChild` (`build.go:47`).**
   - Pinned by none.
   - The stand-in's orphan branch is gated on `!isBuild` (`test_integration_test.go:275`), so no test drives an orphan through the build.
   - If `runBuild` passed a zero delay, the build would hang again and nothing would fail.
   - The decision record's Expected section names the build step explicitly.
6. **`RunGate` passes `defaultPipeWaitDelay` (`rungate.go:28`).**
   - Pinned by none.
   - The orphan subtest goes through `newCommand` in the gate helper, with the delay injected as `GATECLI_TEST_PIPE_DELAY`.
   - `RunGate` is the path of `lym adopt check`, which is where the bug was seen. A zero or missing value there would go unnoticed.
7. **`pipeWaitDelay` plumbing through `newCommand` to `testRequest`, and through the `runnerCmd` rebuild closure (`cli.go:121`).**
   - The `testRequest` hand-off is covered indirectly by the orphan subtest.
   - The detached-runner rebuild path has no test of its own, but it passes the same value. This is minor.
8. **The existing timeout and cancel paths still run `killLeftovers` after `Wait`.**
   - Covered by the unchanged subtest "a tier that outlives timeout_min…" (line 1534) and by the signal subtests.
   - `ErrWaitDelay` is not produced on those paths.
9. **A leftover that closed its output but kept running is also killed.** The summary names this as a design decision.
   - Pinned by none as a separate case.
   - It is the same `killLeftovers` call as behaviour 2, which has no delay-dependent branch. I treat it as covered by behaviour 2.
10. **Log lines (Warn and Info) in `killLeftovers` and the delay path.** Not pinned, and logging does not need a test.

**Findings**

1. **MEDIUM, design:** the `ErrWaitDelay` → `nil` mapping for a zero exit has no test.
   - Location: `child.go:57-61`; the scenario is at `test_integration_test.go:1550`.
   - Evidence: the scenario's `helperExitEnv=134` forces the failing-exit path.
   - A second row, or a table over the exit code (0 and 134) with the orphan, would pin it. Under `PATTERN-test-economy` it should be a row in the same scenario, not a new test.
2. **MEDIUM, design:** the build step's bounded end has no test.
   - Location: `build.go:47`; the stand-in's `orphan := !isBuild && …` at `test_integration_test.go:275`.
   - Evidence: the stand-in explicitly excludes the build invocation from the orphan behaviour.
   - Dropping `e.request.pipeWaitDelay` from the build call would leave every test green.
3. **LOW, design:** `RunGate`'s production wiring of `defaultPipeWaitDelay` is untested.
   - Location: `rungate.go:28`.
   - Evidence: the only orphan scenario enters through `newCommand` with an injected delay.
   - This is the entry point used by `adopt`, the reported failure path.

**Tool calls:** none. The manifest context held every file I needed.
