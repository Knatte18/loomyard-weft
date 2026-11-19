```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per-behaviour list**

1. `slackBridge.RelayPostReady` refuses a strand that is no orch: pinned by `TestSlackBridge_RelayPostReady/AStrandThatIsNoOrchIsRefusedNamingItsOrch`.
2. `RelayPostReady` refuses an orch that has no slack button, such as a pair's orch: pinned by `APairsOrchIsRefusedNamingTheHubsOrch`.
3. `RelayPostReady` refuses an orch whose button is off: pinned by `AnOrchWhoseButtonIsOffIsRefusedNamingTheStatusBar`.
4. `RelayPostReady` counts an orch thread as its worktree's orch: pinned by `AnOrchThreadCountsAsTheOrchOfItsBeam`.
5. `RelayPostReady` answers the relay as running or down from the lock: pinned by `AnOrchWithItsButtonOnWhileTheRelayRunsIsLive` and `...WhileTheRelayIsDownIsNotLive`.
6. `RelayPostReady` passes on a failure of `geometry`, `ForwardingOn` or `RelayLive`: no row. These only propagate errors, so I raise no finding.
7. `Store.slackRelayCheck` with no bridge asks nothing: pinned by `TestStore_PostToTheSlackRelay/AStoreWithoutTheBridgeStoresThePostAndAnswersNoLiveness`.
8. `slackRelayCheck` skips a program sender: pinned by `AProgramsPostAsksTheBridgeNothing`. That row sets both a refusal and a down relay, so the test fails if the check runs.
9. `slackRelayCheck` asks nothing when the receiver is not this hub's relay: **none** (finding 1).
10. `slackRelayCheck` asks the bridge for an EXTERNAL sender (KindExternal): **none** (finding 2).
11. A bridge refusal stores nothing: pinned by `ASenderTheBridgeRefusesIsRefusedWithItsWayForwardAndNothingIsStored`, which checks `errors.Is` and finds no `posted` entry in the journal.
12. A stored post answers `ReceiverLive` and a `Warning` that names `lym slack start` when the relay is down: pinned by the live and down rows of `TestStore_PostToTheSlackRelay`. Both rows compare the warning text exactly.
13. A waiting post sends the relay warning as a progress response: nothing new is needed. This is the generic `awaitOutcome` path, already pinned by `server_test.go:774`.
14. The refusal check runs before `interruptBeforePost`: no row. `--interrupt` to a program is refused by `strandTarget` either way, so swapping the order changes nothing observable. No finding.
15. `creelfake.Bridge.RelayPostReady`, `SetRelayLive` and the shared `Refuse` map are exercised by `TestStore_PostToTheSlackRelay`. The existing users of `Refuse` (`verbs_test.go:448`, `machinemail_test.go:417`) go only through the machine-address post and Reply paths, which never call `RelayPostReady`, so they still mean what they say.
16. The `postCmd` Long text and the doc comments are static prose, which is exempt. The size cap is pinned by `cmd/lym` `TestCLIForm_EveryCommand` (`clitree_test.go:280`).

**Findings**

1. MEDIUM, design. `internal/creelengine/verbs.go:227`. Nothing pins that a post to any other receiver skips the bridge, the `receiver != relay` guard. If the guard is dropped, every strand-to-strand post in a daemon with the bridge calls `RelayPostReady`: a refusal blocks it, or a "relay live" answer overwrites its liveness. No test fails, because:
   - every row of `TestStore_PostToTheSlackRelay` posts to `ly:SLACK` (`machinemail_test.go:310`);
   - the liveness test `PostWith_AnswersWhetherTheReceiverIsLive...` (`verbs_test.go:245–280`) runs with no bridge;
   - the other `withBridge` users do not assert liveness on a local post.

   Fix: add rows to `TestStore_PostToTheSlackRelay` with the bridge set to refuse the sender. One posts to a local strand, and one to a SLACK of another spelling such as a beam's `ly:zz:SLACK`. Each expects the post stored with the strand's own liveness, or nil.
2. LOW, design. `internal/creelengine/verbs.go:223`. No row covers an EXTERNAL sender, though `creelengine/doc.go` and the commit message both name EXTERNAL as a sender the bridge is asked about. If the filter were `sender.Kind != KindStrand`, which drops EXTERNAL, no test would fail. EXTERNAL is `KindExternal` (`seam.go:156`). Fix: add a row `AnExternalPostIsAskedAndRefused` with `external()` as sender, a bridge refusing `ly:EXTERNAL`, and nothing stored.

Tool calls:
- Grepped the `ExternalSender` Kind, the `withBridge` callers and the `ReceiverLive` assertions in tests. My context did not show which tests run with a bridge or assert liveness.
- Read the head of the liveness test in `verbs_test.go` and `newEnv`, to confirm that test runs without a bridge. My context lacked that test's setup.
