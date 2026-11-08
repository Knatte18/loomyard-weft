# Fixer-report, code review round 1

## Changes

- F1: `endLiveRelays` (`internal/slackcli/cli.go`) no longer returns at the first failed stop.
  It collects each failed hub with its error and ends the rest.
  It then returns one error naming every hub not ended with its error and every hub ended, the tokens stored, and `lym slack stop` then `lym slack start` as the way forward.
  Fixed differently inside the proposal: `relays_ended` is filled only when every relay ended, since `runVerb` discards fields on an error; the ended hubs are named in the error text instead.
  Test: the Login row `AFailedStopOfOneHubStillEndsTheOthersAndNamesBoth`, with a `stopErrs` knob on the harness.
- F2: `login` refuses an empty `bots.info` or `hello` app id through `appIDMissingRefusal`, which names the token kind and says nothing was stored.
  Tests: the `AHelloFromAnotherApp` test became the `appRefusals` table with rows for an empty `bots.info` and an empty `hello` app id.
- F3: `TestLiveHubGeometries` in `internal/slackcli/wiring_geometry_test.go` drives `liveHubGeometries` over a temp machine record.
  It is not parallel and says why.
  Rows: one geometry per live hub; an unreadable record, naming the seam's own wording with the record's path and wrapping `state.ErrDecode`; a hub whose geometry fails, named by its path (skipped on Windows, where the token and state directories share `LOCALAPPDATA`).
- F4: the `Socket.Ack` comment states that Slack sends an unacknowledged envelope again, with no deadline.

Commits: F3, F4, then F1 and F2 together (they share `cli.go` and `cli_test.go`), then one commit for the self-check changes.

## Gate repair

The verify gate failed two `cmd/lym` tests, `TestTierPurity_UntaggedTestsSpawnNothing` and `TestTierTags_OneTierPerTestFile`.
The cause: F3's tier-0 test sat in `wiring_test.go`, beside a tier-1 test that uses `hubforge.CopyHub`.
Repair: `TestLiveHubGeometries` moved to its own file, `internal/slackcli/wiring_geometry_test.go`, and `wiring_test.go` is back to its tier-1 content.
`lym gate test ./internal/slackcli ./cmd/lym` passes at tier 0.
The `internal/pairteardown` tier-2 failure in the same verify log is a `/tmp/.git` stray reported by tmuxkit with every pairteardown test passing; no file of this round touches that package or writes outside `t.TempDir`, so it is not repaired here.

A second gate stop followed: the file header of `wiring_geometry_test.go` reached the 120-character limit.
It is now two sentence lines, and the long comment of `machineWithHub` in the same file is split too.

A third gate stop: `TestSocketHolder/ReconnectWaitsDoubleToTheCeilingAndResetAfterAStableConnection` failed once with "no wait of 1s became pending".
This round did not touch `holder.go`; the test raced the holder.
The holder stamps a connection's start after the dial returns, and the test advanced the clock by the stable minute as soon as the dial handed the socket over, so the connection could read as unstable.
Repair: this test and `ARefusedAppMailsOneNoticePerEpisode`, which advances the same way, now wait for the ping wait, which is created after the stamp, before advancing.

## Self-check

- fix-code-correctness, LOW: the unreadable-record row did not pin the seam's own path wording. Fixed as proposed: the row now wants `read the machine record of hubs <path>`.
- fix-code-correctness, LOW: the refusal ended with an empty "relays ended in:" list when no hub was ended, and the `%w` chain on the stop errors was dropped. The empty list is fixed: the ended clause is added only when a hub ended. The `%w` loss is not fixed: the error crosses `runVerb` as text and no caller unwraps it.
- fix-proposal, LOW: the same empty ended list as above. Fixed.
- fix-proposal, LOW: `relays_ended` is dead on the failure path. Fixed in the doc comment of `endLiveRelays`, which now says it is filled when every relay ended.
- fix-proposal, LOW: an absent `hello` app id would make every login refuse. Not changed: F2 asks for exactly this refusal, and storing tokens of unproven app is the defect F2 names; the refusal's way forward names the tokens to check.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
