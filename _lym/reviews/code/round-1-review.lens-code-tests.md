```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. Each new `transportPhrases` entry is classified as a transport failure for push, fetch and ls-remote: `TestIsTransportFailure` has one phrase row per entry (`internal/gitexec/transport_test.go:28-55`). The exception is the sideband row (`:49`), whose stderr holds both `unexpected disconnect while reading sideband packet` and `early eof`. Removing either phrase alone still passes that row, so neither phrase is pinned on its own.
2. A 403 or a 404 is not a transport failure: rows "forbidden fetch" and "missing repository ls-remote" (`transport_test.go:66-67`).
3. A network-failure stop below the cap re-steps after 15 s, then 30 s, then 1 min: `TestDecideDrive` rows (`driverules_test.go:88-90`), which compare the wait against literals.
4. A network-failure stop past the cap hands back with `NetworkNotice`: `TestDecideDrive` row at `driverules_test.go:91`.
5. A changed build hands over before the network rows: `TestDecideDrive` row at `driverules_test.go:92`.
6. Other transient classes still go to the tackler: `TestDecideDrive` row at `driverules_test.go:87` (agent-start). The host-resources class has no row, but it takes the same branch.
7. `networkFailure` treats a GitHub API failure as a network failure: only the RepairCount 2 row (`driverules_test.go:90`). That row would fail if GitHub API were dropped from the switch, so this behaviour is pinned.
8. `settleStop` calls `Deps.Wait` with the decided delay before each re-step, and the four-step run ends in a hand-back with no tackler call: `TestGoDriver_Settle` row "a network failure that outlasts every wait…" (`drive_test.go:311-324` in the diff). It asserts the calls `wait:15s`, `wait:30s` and `wait:1m0s` in order, three re-step records, one hand-back, and no tackler.
9. The exact notice text from `networkNotice` (row, re-step count, total wait, quoted error, resume advice, no question): same `TestGoDriver_Settle` row, `wantNotice` and `wantNoNotice: "Question:"`, built from literals ("3 re-steps over 1m45s").
10. The `wait` real-clock fallback when `Deps.Wait` is nil (the timer, and the early return on ctx end): none.
11. `settleStop` returns the wait's error and exits when the wait ends with an error or a cancelled ctx: none.
12. The wait comes before `holdPause` and `recheck`, so a pause or orch goto made during the wait is honoured before the re-step: none.
13. `networkRestepSpan` sums the waits: covered indirectly by the "1m45s" literal in the notice row (item 9).

**Findings**

- **MEDIUM, design.** `internal/sheddriver/drive.go:982-995`, `goDriver.wait`'s unwired branch. Production runs only this real-clock path: neither the battencli nor the loomcli `Deps` wiring sets `Wait` (`internal/battencli/drive.go:257`, `internal/loomcli/drivewiring.go:236`; the summary says so too). No test reaches it, so its contract is untested: return `ctx.Err()` when ctx ends, and return after the delay otherwise. `Deps.Wait` states this contract at `drive.go:87` in the diff. The test harness wires `Wait` in every row (`drive_test.go:861`). The mutation report's uncovered changed blocks fit this branch. A test with a pre-cancelled ctx and a long delay could pin it without sleeping.
- **LOW, design.** `internal/sheddriver/drive.go:289-293`, the wait-error exit in `settleStop`. No row makes `Deps.Wait` return an error, so nothing checks that a cancelled wait ends the driver with that error and takes no re-step, settle record or hand-back. Evidence: the only `Wait` stub always returns nil (`drive_test.go:861-864`).
- **LOW, design.** `internal/sheddriver/drive.go:289-298`, wait placed before `holdPause` and `recheck`. The summary claims "a pause or an orch goto made during the wait is still honoured before the re-step". No row pins that order: the harness records `hold` only while a pause covers the run (`drive_test.go:819`), and no network-failure row combines a pause or a goto with the wait. If the wait moved after `recheck`, every test would still pass.
- **NIT, design.** `internal/gitexec/transport_test.go:49`, the sideband row. One stderr holds two new phrases, so deleting `early eof` or `unexpected disconnect while reading sideband packet` from `transportPhrases` (`transport.go:35-36`) does not fail the test. Split it into one row per phrase.

No assertion at the review base was widened. The changed `TestDecideDrive` row (`driverules_test.go:234`→`:87`) replaced the non-class string "network" with the real class `agent-start`. It still asserts that the tackler decides.

Tool calls:
1. grep of `drive_test.go` for the harness's call recording and `Wait:` stubs, and of the sheddriver, battencli and loomcli tests for `Wait:` wiring. My context did not show how the test harness records hold and wait calls, or whether any test leaves `Wait` nil.
