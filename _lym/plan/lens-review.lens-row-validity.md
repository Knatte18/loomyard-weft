```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches its branch, fails when the code is wrong, and is not a repeat.

- **BLOCKING** — Cards 10 and 11, D7/D8/D10 test reach. `TestBridgeSend` and `TestBridgeReceive` drive `bridgeSender`, `bridgePoll` and `bridgeReader`, which are all unexported. Every slackengine test lives in the external package `slackengine_test` (glyphs `internal/slackengine_test#…`), and `internal/slackengine/export_test.go` exports only `Inbound`, `Outbound`, `Starter`, `SocketHolder` and `ChannelKeeper` with their methods (export_test.go:19-129). Neither card lists new `export_test.go` aliases or wrappers (e.g. `NewBridgeSender`, `Send`, `SettleAccepted`, `NewBridgePoll`, `NewBridgeReader`, `Read`, `Resync`). As written, no row can construct or run the code it claims to reach. Fix: list the export wrappers on each card.

- **MEDIUM** — Card 6, D3 wiring, `TestCreelCLI` rows. The rig does not start its daemon through `serveDaemon`; it calls `creelengine.Run` with its own `Options` (internal/creelcli/cli_test.go:347-354, `Hubs: hubs` and no `Bridge`). Without `Options.Bridge`, the "stubbed to ready" rows reach `postMachine`'s nil-bridge refusal instead of the take, settle and deliver path. The card lists no edit to the rig's daemon start or a new rig field. `serveDaemon`'s one new line, `Options.Bridge = newSlackBridge(hubPath)`, is also reached by no row.

- **MEDIUM** — Card 9 (and 11), the nil seam of the new `Hub.MachineNames`. `relay_test.go:119` builds a `slackengine.Hub{…}` literal of its own in `newRelayRun`, apart from `fakeHub.hub()` (inbound_test.go:636). The card adds `MachineNames` and edits only `fakeHub`. Every `TestRunRelay` run then has a nil `MachineNames`, which panics once the approve, status or bridge-reader code calls it. The card states no meaning for a nil field and does not list `newRelayRun`.

- **MEDIUM** — Card 3, D3 criterion "no per-hub outbound worker starts at the scheduler's start or on a post". The planned row asserts no spawn, no deliver and no `remote-delivered` or `outbound-returned` record. That would not fail if a machine-outbound message wrongly entered the per-hub map: its empty `ReceivingHub` makes `attempt` fail at `hubs.Geometry` (outbound.go:324) with `codeHubUnreadable`, so nothing is spawned or delivered, and no record appears until the hour passes. Fix: assert that `creelfake.Hubs` saw no `Geometry` call for any hub, or that the outbound map stays empty, or advance the clock past the hour.

- **MEDIUM** — Card 9 and card 8, D6 `approve <machine> <app id>`. The command depends on `BotUsersNamed` with an empty name keeping every bot user. Card 8's `TestAPI` rows cover only name matching, so the empty-name branch of the real API is reached only through `fakeSlack`, and only by its own reimplementation.

- **LOW** — Card 9 rewords the `status` answer to `presence is %s; this machine's app id is %s`. Existing `TestInboundRead` asserts the old exact answers `presence is slack` and `presence is terminal` (inbound_test.go:1293, :1415), and the Intent does not say those rows change. A missing credentials file gives an empty app id, and the card states no wording for that.

- **LOW** — Card 10, the D4 row "an `already_in_channel` answer clears the hold". `already_in_channel` is mapped to nil inside `API.Invite` (api.go:320), so `fakeSlack` cannot produce it meaningfully. The row reduces to "an invite fails, then succeeds", and the code path it names lies in `API`, which no row covers.

- **LOW** — Card 3, test economy. The three `TestStore_Mail` refusal rows for button-off, missing token and pair's orch all reach the same branch, where `bridge.SendReady`'s error is returned as it is, through the same fake. They repeat each other, and the real refusals are covered by card 6's `TestSlackBridge`; one row is enough.

- **LOW** — Card 3, Uses. The start-scan row runs `env.start` (a deliver_test.go helper), which is not listed in the card's Uses.

- **LOW** — Card 4, dispatch coverage. The `machine-report` dispatch branch in `daemon.dispatch` is reached by no row: card 4 calls the Store methods directly, and card 6's `TestCreelCLI` rows cover only take, settle and deliver.

- **LOW** — Card 11, the `TestSocketHolder` row asserts "rereads no orch channel". The holder only writes hints; reading is the inbound's job. The holder test can only check the hint's name (no `resync` hint), so that part of the assertion cannot fail there.

Tool calls:
1. Grepped `fakeSlack`'s methods, the `export_test.go` declarations, the `Hub{` literals in tests and the `presence is` assertions. My context did not hold `export_test.go`'s full list or the other places that build a `Hub`.
2. Read the creelcli rig's daemon start and `fakeSlackAPI` (api_test.go:25-80). My context did not show how the rig fills `creelengine.Options` or how `fakeSlackAPI` answers.
