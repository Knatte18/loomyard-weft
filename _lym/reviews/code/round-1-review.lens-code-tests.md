```yaml
lens: code-tests
outcome: reported
```

I reviewed the diff's tests under the code-tests lens: for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Behaviour → pinning test**
1. A strand's post to `ly:SLACK` with the relay live is stored and answers `receiver_live: true`: `TestStore_PostToTheSlackRelay/AnOrchsPostWhileTheRelayRunsIsStoredAndLive`.
2. Relay down: the post is stored, answers `receiver_live: false` and carries the exact `relayDownWarning` text: `TestStore_PostToTheSlackRelay/AnOrchsPostWhileTheRelayIsDownIsStoredWithAWarningNamingSlackStart`.
3. A bridge refusal is returned and nothing is journaled: `TestStore_PostToTheSlackRelay/ASenderTheBridgeRefusesIsRefusedWithItsWayForwardAndNothingIsStored`.
4. A program sender asks the bridge nothing: `TestStore_PostToTheSlackRelay/AProgramsPostAsksTheBridgeNothing`. The fake is set to refuse and to report the relay down, so asking would fail the row.
5. A daemon without the bridge stores the post and answers no liveness: `TestStore_PostToTheSlackRelay/AStoreWithoutTheBridgeStoresThePostAndAnswersNoLiveness`.
6. `slackRelayCheck` asks only when the receiver is this hub's relay program: **none**.
7. An EXTERNAL sender's post to `ly:SLACK` asks the bridge and is refused as "no orch": **none**.
8. The relay check runs before `interruptBeforePost`, so a refused post stops no turn: **none**.
9. A waiting post to `ly:SLACK` carries the relay warning in a progress response: no relay-specific row. `server_test.go:774` pins how `awaitOutcome` sends a stamp's warning, and that code is unchanged. Not a finding.
10. `slackBridge.RelayPostReady` answers live and down for an orch whose button is on: `TestSlackBridge_RelayPostReady/AnOrchWithItsButtonOn…IsLive` and `…IsNotLive`.
11. An orch thread maps to its beam's orch: `TestSlackBridge_RelayPostReady/AnOrchThreadCountsAsTheOrchOfItsBeam`.
12. The button-off refusal: `TestSlackBridge_RelayPostReady/AnOrchWhoseButtonIsOffIsRefusedNamingTheStatusBar` (substring check).
13. The no-button refusal for a pair's orch: `TestSlackBridge_RelayPostReady/APairsOrchIsRefusedNamingTheHubsOrch` (substring check).
14. The non-orch refusal: `TestSlackBridge_RelayPostReady/AStrandThatIsNoOrchIsRefusedNamingItsOrch` (substring check).
15. `RelayPostReady` passes on errors from `geometry`, `Format`, `hasButton` and `ForwardingOn`: **none**. These are plain pass-throughs of internal calls, not new trust-boundary behaviour, so no finding.
16. In `creelfake.Bridge`, `Refuse` now covers `RelayPostReady`, and `SetRelayLive` sets the relay state: rows 2–4 of `TestStore_PostToTheSlackRelay` use both.
17. The help text and doc comments are static prose and exempt. The help-size cap is the gate's (`cmd/lym/clitree_test.go:280`).

**Findings**

- **MEDIUM**, class `design`, `internal/creelengine/machinemail_test.go:285` (`TestStore_PostToTheSlackRelay` table)
  - Summary: no row posts as EXTERNAL. The summary and `creelengine/doc.go` say an EXTERNAL post to `&lt;shortname&gt;:SLACK` asks the bridge, and `slackRelayCheck` excludes only `KindProgram`. Nothing would fail if EXTERNAL were treated like a program and skipped the check.
  - Gap: `TestSlackBridge_RelayPostReady` (`internal/creelcli/slackbridge_test.go:33`) also has no `ly:EXTERNAL` row to pin its "no orch" refusal.
  - Evidence: `internal/creelengine/verbs.go:223`; `internal/creelengine/seam.go:151` (EXTERNAL is `KindExternal`).

- **LOW**, class `design`, `internal/creelengine/verbs.go:193`
  - Summary: the diff moves the relay check ahead of `interruptBeforePost` so a refused post interrupts nothing, but no row posts to `ly:SLACK` with `Interrupt`. Moving the check back after the interrupt would pass every test.
  - Evidence: `internal/creelengine/verbs.go:193-203`; the test's `PostOptions{}` at `internal/creelengine/machinemail_test.go:314`.

- **LOW**, class `design`, `internal/creelengine/verbs.go:226-228`
  - Summary: nothing pins the receiver gate (`receiver != relay` returns nil).
  - Why nothing catches it: every `TestStore_PostToTheSlackRelay` row posts to `slack`. The existing liveness rows (`verbs_test.go:280`, `server_test.go:752`) run without a bridge. If the check ran for every receiver, a post to a dormant strand with the bridge set would get `receiver_live: true` and lose its warning, and nothing would fail.
  - Evidence: the `withBridge` uses listed by grep. None assert `ReceiverLive` on a strand receiver.

- **NIT**, class `consistency`, `internal/creelcli/slackbridge_test.go:72-74`
  - Summary: the refusal rows check a substring of the way-forward clause, not the whole message. The `scribe:testing` skill prefers exact equality, and a reworded state clause ("is no orch", "has no slack button", "is off") would still pass.
  - Evidence: `strings.Contains(err.Error(), tt.wantErr)` at `slackbridge_test.go:73`.

**Tool calls**
1. Grepped `ReceiverLive`/`withBridge` in the creelengine tests and read `ValidateRole`. My context lacked which existing tests check liveness with a bridge set, and how an `EXTERNAL` role parses.
