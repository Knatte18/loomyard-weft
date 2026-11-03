```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: behaviour → pinning test**

Hibernate (`hibernateengine`, `hibernatecli`, `beamcli`, `fabriccli`):
- **Carrier takeover:** `TestTakeUp` in `carrier_test.go` pins it, with four rows for no daemon, a dead daemon, the process itself, and the pausing and confirming phases. The hibernating and resuming phases and the failed lift are covered too.
- **`Carrier.Tick`, `HoldsReexec`, wake-touch:** `TestCarrier` in `carrier_test.go` covers all three.
- **Submit without spawn, `SubmitReport`, dobby way forward:** `TestSubmit` in `verbs_test.go`. I read only the test function names, not the rows. Main authored it, and the merge diff equals main's diff.
- **Submit refusing any pending request, and Cancel without a kill:** `TestSubmit` and `TestCancel`, with the same caveat.
- **`TerminalPhases` matching `fabriccli`'s ended phases:** `TestHibernateEndedPhases_AreTheEnginesTerminalPhases` in `fabriccli/hibernatedaemon_test.go`.
- **`hibernateDaemonLive` skipping ended phases:** `TestHibernateDaemonLive` in the same file, one row per phase.
- **`beamcli` watching the request dir only for the prime:** `TestDobbyRuns_PairAtItsOwnAnchor` in `dobbypairanchor_test.go` covers both the listed and the empty path.
- **Prime Tick set, beam Tick nil, request dir created:** `TestHubDobbyDeps_PrimeTidyShuttle` in `dobbydeps_integration_test.go`.
- **`holdReexec`:** `TestHoldReexec` in `dobby_test.go`.
- **`hibernatecli` dropping the `watch` verb and the spawn/kill code:** no test asserts the verb is gone. It is a removal, and a leftover reference would fail the build, so I don't count it a gap.
- **`hibernatecli.NewDobbyCarrier` and `RequestPath`:** `RequestPath` is exercised through the `beamcli` and `fabriccli` tests above. `NewDobbyCarrier` is exercised only through `TestHubDobbyDeps_PrimeTidyShuttle`.

Drift, gate and config:
- **`gateslot.ResolveCommandDir`:** `TestResolveCommandDir`.
- **`gateslot.StatCommandDir`:** `TestCheckDirs` and `TestRunCapture` in `driftengine/capture_test.go`, both indirectly. No direct test of `StatCommandDir`'s message, but the capture and compare tests assert its text through the refusal.
- **`gateslot.CheckDirs`:** `TestCheckDirs` in `gateslot/config_test.go`.
- **`driftengine.CheckDirs`:** `TestCheckDirs` in `driftengine/config_test.go`.
- **The drift step's empty-dir default, no longer `"."`:** `TestLoadConfig` and `TestConfigSpec_*`.
- **Capture, side capture and compare dirs, including missing-dir refusals and way forwards:** `TestRunCapture` and `TestRunCompare`.
- **`captureReading` with `AnchorDir`:** `TestSeatTables`, "the capture's run directory is told relative…" and the `AnchorDir` required-field row.
- **`driftcli` `captureRequest` and `compareRequest`:** `TestWorktreeRequests_JoinOnTheWorktreeRootAndStartInTheAnchor`.
- **`loomcli` drift request bases:** `TestDriftDeps_RunWhereTheStepsDirResolves`, integration tier. I did not look for a row checking `BaselineInputs.AnchorDir` at the `loomcli` wiring. `TestSeatTables` covers the engine side only.
- **`configcli` hub-wide `CheckDirs` on set and on editor save:** rows added in `configcli_test.go`, covering drift set refused, drift set accepted, gate build dir refused, editor save refused, and editor save over an already-failing file written.
- **`configcli` gate `tiers.<key>.dir` refusal:** no row. Only a gate build dir row exists.
- **`battencli` `PairDir` now the worktree root:** `stepPairDirNamesTheTaskWorktree` and `readNoticeChild`, both made anchor-sensitive with `ReanchorPair`.
- **`configreg` `CheckDirs` field set for drift and gate:** exercised through the `configcli` rows.
- **`lymdirs`, `cleanupcli` and `doc.go` changes:** comment or text only.

**Findings**
1. **LOW, `design`.** `internal/configcli/configcli_test.go` has no row where a hub-wide gate write of a `tiers.<key>.dir` is refused for a missing dir. `gateslot.CheckDirs`'s tier loop is pinned only at the engine level by `gateslot/config_test.go` `TestCheckDirs`, not through the hub-wide write that wires it. The added `configcli` rows name only a gate build dir. I did not read the gate `TestCheckDirs` rows, so I can't say whether they cover the tier path. Evidence: `git diff` of `configcli_test.go` and `gateslot/config_test.go:167`.

I found no redundancy to report. I did not compare new test rows against existing ones beyond the names above.

**Tool calls (3)**
1. Read the `carrier_test.go` function and row names, and the `hibernateengine` `verbs.go`, `request.go` and `wake.go` diffs. My context held `carrier.go` and `watch.go` but not the tests or the Submit and Cancel changes.
2. Read the `configcli`, `configreg`, `battencli`, `cleanupcli` and `lymdirs` diffs, and the `configcli` test row names. My context lacked these.
3. Read the `battencli` test diff, the grep for the gate, drift and hibernate test function names, and the grep for `CheckDirs` in tests. My context lacked the test locations.
