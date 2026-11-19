```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Per behaviour, the test that pins it**
- **`slackBridge.RelayPostReady`, a strand that is no orch is refused with its way forward:** `TestSlackBridge_RelayPostReady/AStrandThatIsNoOrchIsRefusedNamingItsOrch`. An EXTERNAL sender takes the same `!IsOrchStrand` branch and has no row of its own.
- **`RelayPostReady`, an orch thread maps to its worktree's orch:** `/AnOrchThreadCountsAsTheOrchOfItsBeam` covers the beam case. A thread of the prime orch takes the same `TaskSlug` path and has no row.
- **`RelayPostReady`, an orch with no button (a pair's orch) is refused:** `/APairsOrchIsRefusedNamingTheHubsOrch`.
- **`RelayPostReady`, an orch whose button is off is refused:** `/AnOrchWhoseButtonIsOffIsRefusedNamingTheStatusBar`.
- **`RelayPostReady`, live and down answers:** `/AnOrchWithItsButtonOnWhileTheRelayRunsIsLive` and `/…WhileTheRelayIsDownIsNotLive`.
- **`RelayPostReady`, a failed read of the geometry, the beam listing, the forwarding file or `RelayLive` is passed back as the refusal:** no row.
- **`Store.post` + `slackRelayCheck`, a refused sender stores nothing:** `TestStore_PostToTheSlackRelay/ASenderTheBridgeRefusesIsRefusedWithItsWayForwardAndNothingIsStored`. It checks `errors.Is` and counts zero `posted` entries.
- **`post`, the relay state sets `ReceiverLive` and the exact warning:** `/AnOrchsPostWhileTheRelayRunsIsStoredAndLive` and `/AnOrchsPostWhileTheRelayIsDownIsStoredWithAWarningNamingSlackStart`. The second pins the full `relayDownWarning` text.
- **`slackRelayCheck`, a program sender is not asked:** `/AProgramsPostAsksTheBridgeNothing`. The bridge refuses `batten`, so asking it would fail the row.
- **`slackRelayCheck`, a store without a bridge is not asked:** `/AStoreWithoutTheBridgeStoresThePostAndAnswersNoLiveness`.
- **`slackRelayCheck`, an EXTERNAL sender is asked like a strand:** none.
- **`slackRelayCheck`, a receiver other than the relay program asks nothing and keeps its own liveness:** none.
- **`post`, the relay check runs before `interruptBeforePost`:** none. The order is barely observable, since `strandTarget` refuses an interrupt of a program anyway. No finding.
- **The relay state rides the waiting-post path (pending `ReceiverLive`, progress `Warning`):** this path is generic and the diff did not change it, so it is not a new behaviour. No finding.
- **`creelfake.Bridge.RelayPostReady`, `SetRelayLive`, and `Refuse` keyed for both methods:** exercised by `TestStore_PostToTheSlackRelay`. These are test support, not a product behaviour.
- **`postCmd` Long text, the doc comments and the file headers:** static prose, exempt.

**Findings**

1. MEDIUM, class `design`. Location: `internal/creelengine/machinemail_test.go`, `TestStore_PostToTheSlackRelay`. Summary: no row pins that a post to a receiver other than `<shortname>:SLACK`, sent while a bridge is set, skips the bridge and keeps its own liveness. If the `receiver != relay` guard in `slackRelayCheck` were dropped or inverted, every local post in a daemon with the bridge would have its `ReceiverLive` and `Warning` overwritten by the relay state, or be refused by the bridge. No current test would fail. Evidence: the guard is at `internal/creelengine/verbs.go:226-229`. The tests that set a bridge and then post locally assert neither liveness nor a refusal: `verbs_test.go:1715-1718` (`ClaimedLineOfMachineMail…`, which posts alpha→orch and checks only the claimed lines), `deliver_test.go:301` and `deliver_test.go:1486`. Every row of the new test posts to `slack`. Fix: add a row that posts to a strand while the bridge refuses the sender or reports the relay down, and want it stored with the strand's own liveness.

2. LOW, class `design`. Location: `internal/creelengine/machinemail_test.go`, `TestStore_PostToTheSlackRelay`. Summary: no row pins that an EXTERNAL sender is asked by the bridge. The engine skips the check only for `KindProgram` (`verbs.go:222`). If that were widened to "anything not a strand", EXTERNAL posts to SLACK would go back to silent `ok`, and no row would fail. The bridge-side refusal of EXTERNAL is reached only through the generic non-orch row (`slackbridge_test.go:166`, `ly:pay:driver`). Fix: add an `external()` row with the bridge refusing that sender, and want the refusal with nothing stored.

3. LOW, class `design`. Location: `internal/creelcli/slackbridge_test.go`, `TestSlackBridge_RelayPostReady`. Summary: no row reaches a failed bridge read. That covers `geometry()`, the `beams()` listing (only reached for a non-prime orch), `ForwardingOn`'s read error and `RelayLive`'s error. Each is passed back as the post's refusal, which is the behaviour the diff newly attaches to `lym creel post` to SLACK. Evidence: `slackbridge.go:97-99`, `hasButton` reaching `b.beams()` at `slackbridge.go:133-136`, `slackbridge.go:112-115` and `slackbridge.go:120`. The table's `geometry` and `beams` closures always answer nil errors (`slackbridge_test.go:62-63`). Fix: add a row whose `beams` (or `geometry`) answers an error, and want that error back.

Tool calls:
1. `grep withBridge` across the `creelengine` tests: to find which existing tests post with a bridge set. The manifest held only two of them.
2. Read of the four tests that set a bridge (`verbs_test.go:380-400`, `verbs_test.go:1708-1735`, `deliver_test.go:295-320`, `deliver_test.go:1480-1500`): to check whether any of them asserts liveness or a refusal on a non-SLACK local post. None was in context.
