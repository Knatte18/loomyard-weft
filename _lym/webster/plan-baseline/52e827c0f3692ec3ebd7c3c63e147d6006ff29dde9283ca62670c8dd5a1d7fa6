# Card 6 — creel-bridge-wiring

**Create:**
- `internal/creelcli#GoClient.TakeMachine`
- `internal/creelcli#GoClient.ReportMachine`
- `internal/creelcli#GoClient.SettleMachineDelivered`
- `internal/creelcli#GoClient.SettleMachineReturned`
- `internal/creelcli#GoClient.DeliverMachine`
- `internal/creelcli#slackBridge`
- `internal/creelcli#newSlackBridge`
- `internal/creelcli#slackBridge.Names`
- `internal/creelcli#slackBridge.SendReady`
- `internal/creelcli#TestSlackBridge`

**Edit:**
- `internal/creelcli#serveDaemon`
- `internal/creelcli#messageFields`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#rig`
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#trustField`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `internal/creelengine#MachineBridge`
- `internal/creelengine#VerbMachineTake`
- `internal/creelengine#VerbMachineReport`
- `internal/creelengine#VerbMachineSettle`
- `internal/creelengine#VerbMachineDeliver`
- `internal/slackengine#ForwardingOn`
- `internal/slackengine#RequireConfigured`
- `internal/machinename#Names`
- `internal/creelcli#NewProgramClient`
- `internal/creelcli#writeMessagesText`

**Intent:**
The wiring half of D3: the Go-client calls the relay makes, and the `MachineBridge` creel's daemon is told, filled from `internal/slackengine`.

Each new `GoClient` method sends its verb under `goCallTimeout` and answers what the response carries (`MachineOutbound`, or `ID` for a deliver); a daemon refusal comes back as an error that `creelengine.IsDaemonRefusal` matches.
No main reaches them until the relay calls them through its `Mailbox`, so `cmd/lym#testOnlyCodeAllowed` gains `internal/creelcli.GoClient.TakeMachine`, `.ReportMachine`, `.SettleMachineDelivered` and `.SettleMachineReturned`, each with the reason `no caller until card 10's relay sends machine mail; that card removes this entry`, and `internal/creelcli.GoClient.DeliverMachine` with the reason `no caller until card 11's relay delivers bridge mail; that card removes this entry`.

`slackBridge` is creel's `MachineBridge` over the hub at hubPath, built by `newSlackBridge` from three function fields a test replaces: `geometry` (`hubgeom.SlackGeometry`), `beams` (the hub's beam slugs, read as `slackcli`'s `beamSlugs` reads them, through `hubgeom.DobbyStart` on the prime) and `names` (`machinename.Names`).
Every call reads afresh, so a pressed button or a new token applies at the next post.
`Names` answers `names()`.
`SendReady(orch)` refuses, in order:
1. An orch that is neither the prime's (`<shortname>:orch`) nor a listed beam's: `creel: %s has no slack button, so it posts to no other machine; way forward: ask the hub's orch or a beam's orch to post it`. This is the pair's-orch refusal of D2.
2. A button that is off (`ForwardingOn` false): `creel: the slack button of %s is off, so its mail cannot cross machines; way forward: press the button on its status bar`.
3. `RequireConfigured(geom)` refusing, returned as it is: it names `lym slack login` or `lym slack setup --operator <user-id>`.
A failed geometry, beams or forwarding read is returned as it is.
`serveDaemon` sets `Options.Bridge` to `newSlackBridge(hubPath)`, so creel takes a post across machines from here on.

The JSON writers show machine mail as they show other-hub mail: `messageFields` emits `trust` (through `trustField`), `sending_id` and a `sending_machine` field for a message whose `SendingMachine` is set, and `logEntryFields` adds `receiving_machine`, `sending_machine` and `slack_ts` beside `sending_hub` and `receiving_hub`.
`trustField`'s doc comment becomes: the trust value of a message from another hub or another machine.

`TestSlackBridge` (tier 0) builds a `slackBridge` over a temp Slack geometry and stubbed beams, and is table-driven over orch and files: the prime orch with its button on and both tokens and a setup is ready; a beam orch likewise; a pair's orch is refused naming the hub's orch; a button off, a missing app token and a missing setup are each refused naming their way forward.
`rig` gains a `bridge creelengine.MachineBridge` field, a `slackBridge` whose function fields are stubbed to ready, and `rig.startDaemon` passes it as `Options.Bridge`, so the rows below reach the bridge path the daemon serves.
`TestCreelCLI` (tier 1) gains rows over that rig:
- a program client signing `SLACK` takes an orch's post to `osl::zz:orch`, reports a `peer-unapproved` hold, settles it delivered, and `lym creel log --id --json` shows `machine-held` and `machine-delivered` with `receiving_machine` `osl` and its `slack_ts`;
- the same client delivers mail from `osl::zz:orch` to the hub's orch, the orch's `lym creel read` header shows `"osl::zz:orch" (other machine, trusted orch)`, and `lym creel read --json` shows its `trust`, `sending_id` and `sending_machine`.
`internal/creelcli/doc.go` gains the machine verbs of the Go client, the bridge the daemon is told, and the machine fields of the read and log JSON.

**ImpactSummary:** `serveDaemon` gains one option; four of the new client methods have no caller until card 10, and `DeliverMachine` none until card 11; `internal/creelcli` now imports `internal/slackengine` and `internal/machinename`, which import no `creelcli`, so no cycle forms.

**Commit:** 6: creel's Go client and daemon reach the Slack bridge
