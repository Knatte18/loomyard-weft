```yaml
lens: code-tests
outcome: reported
```

Task: map each behaviour the diff adds or changes to the test row that pins it, or none.

**Per-behaviour list**

| Behaviour | Test that pins it |
|---|---|
| Marks in order: starting, each tier in flight, each tier ended, green or failed end, with the D2 texts | `TestRunCheckMarks` (check_test.go): green, failed tier, failed before any tier |
| Start time of the tier in flight is the tier's own start | Weakly pinned. See F1. |
| A mark that fails leaves the record and the notices unchanged | Weakly pinned. See F2. |
| `suiteMessage` for started, with tiers, and for found running | `TestAdopt` rows `adopt check started ... tiers 0, 1, 2` and `CheckFoundRunningWithNoTiersReadIsNamedAlreadyRunning` |
| No `message` on a failed start, `suite_warning` kept | `TestAdopt` row `FailedSuiteStartIsTheWarningNamingTheGate` |
| `ensureCheck` sets `AlreadyRunning` only for a check found running | `TestEnsureCheck` `wantRunning` rows |
| The envelope emits `message` as a JSON field (cli.go:105) | None. My grep found no test on the `"message"` key or on `suite_warning` in the adoptcli tests. `TestAdopt` asserts only the `outcome` struct. |
| `adoptGateTiers`: tiers carried on `suiteStart`, and omitted when the gate config does not read or has no `adopt` key | None. My grep found no reference to `adoptGateTiers` in any adoptcli test. Nothing exercises the `StartSuite` wiring in deps.go that sets `Tiers`. |
| `hubCheckDeps` `Mark` and `primeReed`, including a reed config or geometry error returned as the mark's error | None. `check_integration_test.go` overrides `deps.Mark` with a no-op, so the real wiring never runs. |
| `SetCheckMark` set and unset argv | `TestSetCheckMark` (checkmark_test.go): three rows |
| `SetCheckMark` only when the session is up | None. No row uses a fake tmux with no session, and no row asserts that no `set-option` is issued. |
| Line 2 renders the mark with its text, seconds, state styling, no seconds once ended, and gone when cleared | `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` (tier2, real tmux) |
| Mark sits before the clock | Weakly pinned. See F5. |

**Findings**

- **F1, LOW, design**, check_test.go:360-420 (`TestRunCheckMarks`) and check_test.go:47. The fixture clock is the constant `checkFixtureNow`, so `record.Start` and `tierStart` are equal. A regression that timed the tier in flight from `record.Start` instead of `tierStart` would still pass. Fix: advance the fixture clock per call and assert distinct `Start` values on the tier-in-flight marks.
- **F2, LOW, design**, check_test.go "a notice or mark that fails changes nothing in the record". The row fails `Notice` and `Mark` together. It cannot show that a failing mark leaves the notices posted, nor that a failing notice leaves the marks set. Fix: add a row, or split the row, where only `Mark` fails, and assert that all notices still posted.
- **F3, MEDIUM, scope**, deps.go `adoptGateTiers` and the `StartSuite` tiers wiring (deps.go:~150, ~238). This is D1's acceptance criterion that a gate config that does not read, or has no `adopt` key, leaves the tiers out. No test fails if the tiers are never carried, or if the missing-key path panics or returns garbage. Fix: add a tier0 test of `adoptGateTiers` over a fixture prime with an `adopt` key, a gate config without the key, and a gate config that does not read.
- **F4, LOW, scope**, cli.go:105. The envelope's `message` field is never asserted. Dropping the `fields["message"]` line would fail no test. Fix: add a case to a cli-level test that decodes the one-line envelope, if one exists, and asserts `message` is present with a started check and absent on `suite_warning`. If none exists, treat as a note only.
- **F5, LOW, scope**, checkmark_test.go. D3 says `SetCheckMark` acts "only when the session is up". The tier0 fake has no row for a missing session. Fix: add a row to `TestSetCheckMark` where the fake reports no session, asserting an error and zero `set-option` argv.
- **F6, LOW, design**, bar_integration_test.go (new test). "Before the clock" is asserted only as "after the follow button" (the `nolist align=right` index comparison). A mark placed after `%H:%M` would still pass. Fix: assert that the index of "adopt check" is less than the index of the clock text.
- **F7, LOW, scope**, hubCheckDeps `Mark` and `primeReed`. The real wiring is untested, so a `reedErr` that is dropped or mishandled would pass. Fix: add a tier0 test of `hubCheckDeps(...).Mark` over a prime whose reed config does not read, asserting that the error is returned.

**Tool calls**
1. A grep over adoptcli tests and checkmark_test.go for `adoptGateTiers`, `primeReed`, `"message"`, `suite_warning`, `requireSessionLocked` and `no session`. The manifest did not show whether tests exist beyond the diff's test files, and it returned nothing.
