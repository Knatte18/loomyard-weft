```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, the test row that pins it, or none.

| Behaviour | Pinning test |
|---|---|
| Takeover of a request no live process carries (no daemon, a dead daemon, the process itself after a re-exec) | `TestTakeUp` row 1, `carrier_test.go:34` |
| Takeover of a pausing or confirming request: owned entry lifted, back to waiting, pauses afresh and ends done | `TestTakeUp` row 2 |
| Takeover of a hibernating or resuming request: ends failed with one notice, entry lifted only with `ResumeOnWake` | `TestTakeUp` row 3, `carrier_test.go:83` |
| Takeover whose lift fails writes nothing | `TestTakeUp` row 4, `carrier_test.go:107` |
| No request, a terminal one, or one another live process carries is left alone | `TestSequence` row `watch_test.go:536`, through `runCarrier` |
| `Carrier.Tick` runs the sequence in a goroutine, starts no second sequence, and `HoldsReexec` is false in the trigger wait, true past it and false once ended | `TestCarrier`, `carrier_test.go:133` |
| `holdReexec` skips the binary check while held | `TestHoldReexec`, `dobby_test.go:118` |
| The prime's dobby has a `Tick`, a beam's has none, and the prime's tick takes nothing up with no request | `TestHubDobbyDeps_PrimeTidyShuttle`, `dobbydeps_integration_test.go:65-74` |
| The watch covers the request file for the prime and not for a beam | `TestDobbyRuns_PairAtItsOwnAnchor`, `dobbypairanchor_test.go:104-107` (covered) and `:128` (no entry for a beam) |
| `Submit` writes a request no daemon carries and reports `DobbyLive` | `TestSubmit` rows renamed in `verbs_test.go` |
| `Submit` names `lym beam start root` when the dobby is down or its liveness does not read | `TestSubmit` row, `verbs_test.go` |
| `Submit` refuses any pending request whether or not a daemon carries it | `TestSubmit` row, `verbs_test.go` |
| `Cancel` stops no process | `TestCancel` rows, `verbs_test.go` |
| A cancel that lands during the pause, before its entry is recorded, has the sequence lift that entry | `TestSequence` row `watch_test.go:1075` |
| The hidden `watch` verb and daemon spawn are gone | `cli_integration_test.go` (operator index and verbs) |
| `RequestPath` and the `DobbyLive` wiring | `cli_integration_test.go:93-97` |
| `fabriccli.hibernateDaemonLive` reports an ended request (done, failed, cancelled) as not live | **none** |
| `refuseLiveDobbies` refusal text for a pending hibernate request | **none** (the anchor test stubs `HibernateLive` at `anchor_test.go:183`) |

**Findings**

1. **MEDIUM, design.** The new ended-phase rule in `hibernateDaemonLive` has no test. Location: `internal/fabriccli/anchordeps.go:240-256`. Nothing in the repo's tests calls it; `anchor_test.go:183` only stubs `HibernateLive`. If the `Phase` check or `hibernateEndedPhases` broke, an ended request would again block `lym fabric anchor --set` on the prime, and no test would fail. It needs a row over a request file each for a pending request with a live carrier, a pending one with a dead or no carrier, and an ended one with a live carrier pid. The summary claims "an ended request no longer blocks it" (`summary.md`, line 30).

2. **LOW, design.** `withdrawn` has one covering row, and it covers only the confirming-advance path. Location: `internal/hibernateengine/watch.go:163-173`. The `watch_test.go:1075` row covers a cancel landing before `Pause` records its entry. The other `withdrawn` callers have no row where the sequence, not `Cancel`, must lift `kept`. Those callers are `confirmationStopped`, the `advance(PhaseHibernating)` refusal and the `resuming !ok` branch. Cancel's own lift covers a recorded entry, so a broken `withdrawn` could not be seen there. One row per path, or one shared row, would pin it.

3. **LOW, design.** The prime-only wrapping of the binary check is not asserted. Location: `internal/beamcli/dobbydeps.go:197-199`. The `Tick` wiring is checked at `dobbydeps_integration_test.go:65-74`, but `deps.CheckBinary = holdReexec(...)` is not. `TestHoldReexec` covers only the helper. Dropping the wrapping would let a re-exec cut a running sequence with no test failing. The integration test could assert that `primeDeps.CheckBinary` differs from the beam's, or that it skips the check while a sequence is held.

No finding for the rest: every other changed behaviour has a pinning row above.

**Tool calls**
1. Read `carrier_test.go` lines 80-189, because my context held only its head.
2. One `grep`/`sed` call over `internal` for `hibernateDaemonLive`, `HibernateLive`, `TestHoldReexec`, the hibernatecli integration test and `watch_test.go`'s cancel rows, because the test bodies were not in my context.
