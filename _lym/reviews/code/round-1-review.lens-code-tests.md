```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, I looked for the test that would fail if it broke. I worked from test names, table-row names and the few spans my greps showed. I opened no test body in full, so every "pinned" below rests on a test name, not a read body.

## Per behaviour: pinning test, or none

| Behaviour | Pinning test |
|---|---|
| Login files both tokens by prefix, with the app id | `Login/StoresBothTokensWithTheAppIDOnlyAfterSlackAcceptsThem`, `OnATerminalPromptsPerTokenAndFilesEachByItsPrefix` |
| Login refuses another prefix, including a user token, and a run without one of each, writing nothing | Table in `cli_test.go` around line 508 (`AUserTokenIsRefused`) |
| Login refuses a `hello` app id that differs from `bots.info`'s | `AHelloFromAnotherAppIsRefusedAndStoresNothing` |
| Login ends every live hub's relay; a failed live-hub read after the write | `Login` cases at `cli_test.go` lines 569-617 (`relays_ended`), `AFailedLiveHubReadAfterTheWriteStoresBothTokensAndNamesTheWayForward` |
| `RequireConfigured` names the missing kind and verb; `start` and `away` refuse | `RequireConfiguredNamesTheVerbThatWritesWhatIsMissing`, `StartAndAwayNeedLoginAndSetup` |
| `EnsureRelay` answers not-configured and spawns nothing | `HubWithNoTokenOrNoSetupIsNotConfiguredAndSpawnsNothing` |
| Relay's missing-token notice, prime channel post, old setup channel fallback | `TestRunRelay/RefusedHubMailsTheOperatorOnceAndExitsWithTheRefusal` (rows at `relay_test.go` lines 317-325 cover the app-level-token-missing case, the old-setup-channel case and the plain prime-channel case) |
| Setup records the operator; `--channel` refused; old `channel_id` kept | `Setup/*` |
| Status lists channels and `last_healthy` | `PrintsPresenceRelayPauseOperatorChannelsAndLastHealthy` |
| `lym slack stop` makes no Slack call (archives nothing) | `Stop/SetsTerminalTellsTheOrchAndEndsTheRelayOnceSparingItself` (line 839) |
| No `conversations.unarchive` anywhere | `keeper_test.go` line 359 reflects over the Slack seam's methods |
| Channel names, hash cut, dated suffix | `TestChannelName`, which includes `ArchivedName` |
| Map cursors forward-only, prune, thread floor, concurrent writes, undecodable map | `TestChannelMap/*` |
| Keeper ensure order, per-orch lock, invite retry and notice | `TestChannelKeeper/*` (rows at lines 122-221 and 365) |
| Keeper `name_taken`: creator test, archived refused, adoption with join, fail-closed reads | `keeper_test.go` lines 221-345 |
| Retire: dated rename, `-2` on `name_taken`, prime never, refused archive retried | `Retire/*` |
| Holder: lock, hint before ack, hint failure, stall limit, unmapped events, live-hub failure, reconnects, backoff, refused app, link disabled, pong stamp | `TestSocketHolder/*` |
| Holder: thread and broadcast hint names | Table at `holder_test.go` lines 645-646 |
| Socket client: hello, ack, ping, refused open, no-hello | `TestSocketClient/*` |
| Outbound routing, `username` and icon, no prefix, gone beam to prime, listing failure | `TestOutboundRead/*` (lines 121-220) |
| Outbound recovery from `is_archived` and `channel_not_found` | Table rows at `outbound_test.go` lines 231-233; the `not_in_channel` row and the once-toward-`refusalCap` count at line 257 |
| Inbound routing by channel, drain, owed reads, rate-limit wait, prune, floor, thread and broadcast once, file lines, presence commands, join on `not_in_channel` | `TestInboundRead/*` |
| Wake loop: file-event wake, one-second floor, own writes wake nothing, on-disk fallback, retry at outbound wake, binary check, holder stop before exec | `TestRunRelay/*` |
| Daemon-wakeups guard lists `holder.go` | `cmd/lym/daemonwakeups_test.go` itself is the guard |
| Geometry fields (`Machine`, `ChannelMapPath`, `InboxDir`, `SocketLockPath`) | `hubgeom_test.go` |

## Findings

1. **LOW, design, test coverage.** I could not find a test for a hub newly named by a `hubregistry.Live` read getting a resync hint (D1; acceptance criterion "as does a hub newly listed").
   - `readLive` (`holder.go:420`, the `liveKnown && !previous[...]` branch) implements it.
   - The only nearby test is `ABlockedLiveHubsReadOwesEveryHubOfTheLastReadAResync` (`holder_test.go:724-743`). It covers the stalled-route owing, not a newly listed hub.
   - `AFailingLiveHubsReadLeavesEventsUnackedAndNoticesOncePerEpisode` (line 768) probably covers the owed resync after a failed read, but not the newly-listed branch.
   - I did not read the 768 body, so this may be covered inside it. The handler should confirm.

2. **LOW, design, test coverage.** I found no wiring test for `liveHubGeometries` (`wiring.go:133`).
   - It builds the live-hub seam from `hubregistry.Live`. D1 and D3 depend on its error text: it must carry the record's path and keep the seam's error as it is.
   - `grep` for `liveHubGeometries` in `slackcli/*_test.go` found nothing.
   - The seam is faked everywhere else, so a regression in its path or error wording would pass every test.
   - Severity is LOW because `wiring_test.go` may exercise it under another name that my grep did not match.

I found no behaviour with a missing pin beyond these two. Nothing else I checked looked uncovered.

## Test economy

I found no duplicated test, no per-helper test that the public surface already covers, and no assertion on static prose. The tests are table-driven and sit at the public surface. I did not read the test bodies, so I cannot rule out a duplicate hidden inside one.

## Tool calls (5 of 6)

1. `grep` of test and subtest names in `holder_test.go`, `keeper_test.go`, `relay_test.go`, `inbound_test.go`, `outbound_test.go`, `socket_test.go`: my context held no test files.
2. Same for `cli_test.go`, `channelmap_test.go`, `channelname_test.go`, `api_test.go`, `files_test.go`.
3. Targeted greps in `cli_test.go`, `holder_test.go`, `keeper_test.go` and `relay_test.go` for `relays_ended`, archive, prime-channel notices and old-setup-channel rows.
4. Targeted greps in `outbound_test.go`, `relay_test.go` and `inbound_test.go`, plus `git diff` of the geometry files, to confirm the recovery rows, the geometry diff and the last-healthy stamp.
5. Listed the slackcli test files, grepped for `liveHubGeometries`, and read `holder_test.go` lines 724-745.
