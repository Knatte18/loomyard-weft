```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that pins it.

**Behaviour → pinning test**

1. **Arrival on exit 5 (no timeout, no crash) passes with no record:** pinned by `TestRedFirstGate_VerdictPerLabelRecordAndExitCode`, row "a check that exits 5, on a hub whose scoper is not go, passes with no record" (`internal/loomshed/redfirstgate_test.go:118`). If the branch were removed, exit 5 would hit `redFirstFailure`'s default case, a terminal failure, and the row's `wantPassed` would fail.
2. **Final evaluation on exit 5 passes:** pinned by the row "a final evaluation of a check that exits 5 … passes with no record" (`redfirstgate_test.go:142`).
3. **Exit 5 together with `TimedOut` is excluded from the pass and fails terminally:** pinned by the row "a check that exits 5 after a tier timeout fails terminally instead of holding" (`redfirstgate_test.go:160`). This row already exists at the merge base.
4. **Exit 5 together with `RunnerCrash` is excluded from the pass:** no test. No red-first row writes the runner-crash line. The `!res.RunnerCrash` conjunct at `redfirstgate.go:216` can be deleted and every test still passes.
5. **`holdsForCommitOrRecord` no longer holds on exit 5:** pinned by rows 1 and 2. A regression would turn row 1's answer pending.
6. **`redFirstHoldLine` collapsed to the no-test wording:** pinned by the exit-4 row's `wantSend` (`redfirstgate_test.go:104`).
7. **`redFirstFailure` drops the exit-5 case:** for a real exit 5 the case is now unreachable, since `verify` passes it first. Exit 5 with a timeout is pinned by row 3.
8. **The hold mechanics, now driven by exit 4:** pinned by `TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands` (`redfirstgate_test.go:306–388`). Its name and doc comment describe a refusal, not a particular exit code. Exit 4 still holds, so the test still checks what it says.
9. **The verb's new exit-5 message ("way forward: none needed, a red-first gate passes this exit"):** pinned by `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged` (`internal/gatecli/redfirst_integration_test.go:153`).
10. **The "passed unjudged" log line in `verify`:** none, and none needed. It is a log field.
11. **Doc comments and the stencil passage:** exempt as static prose.

**Findings**

- **LOW · design** · `internal/loomshed/redfirstgate.go:216`. Nothing tests that exit 5 with a runner crash fails terminally instead of passing, so deleting `&& !res.RunnerCrash` breaks no test. The timeout conjunct has its own row (`redfirstgate_test.go:160`). Evidence: `redfirstgate_test.go:31–211` has no row that writes the gate's runner-crash line. `verifytree/verify.go:440` sets `RunnerCrash` from that line, through `gateslot.FindRunnerCrash` (`verify.go:535`). Fix: add a table row whose command prints the crash line and then runs `exit 5`, expecting a terminal failure that names the runner crash.

**Tool calls**

1. `redfirstgate_test.go` lines 300–420 plus a grep for where `RunnerCrash` is set in verifytree. The manifest stopped at line 300, before the hold test, and did not cover how a crash result is produced.
2. A grep of verifytree and the loomshed tests for `RunnerCrash`. This confirmed where the crash flag is set and that no red-first test uses it.
