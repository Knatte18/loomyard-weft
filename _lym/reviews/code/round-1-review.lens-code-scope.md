```yaml
lens: code-scope
outcome: reported
```

**Task:** code-scope lens. Each diff change is mapped to its decision (D1–D11 and the arbiter's added entry), and each decision to the changes that build it.

## Per change

| Change | Traces to |
|---|---|
| `agentname`: `MachineAddress`, `IsMachineAddress`, `ParseMachineAddress` (card 1) | D1 |
| `creelengine` journal events, `fold.go` pool maps and carry, `record` fields, doc.go (card 2) | D3, D8, arbiter F3 |
| `crossmachine.go`: `postMachine`, `requireOrchSender`, `refuseMachineAddress`; the `refuseOtherHub`, `Forward`, `WaitReply` and `Reply` hooks (card 3) | D2, D3 |
| `machinemail.go`: take, report and settle verbs, `machineHourLoop`; `deliver.go` scheduler hook; `server.go`, `protocol.go`, `client.go`; `outbound.go` `postNoticeLocked` refactor (card 4) | D3, arbiter F1. The `outbound.go` refactor is forced by the machine return notice. |
| `machineinbound.go`: `DeliverMachine`; `HubStamp`; `settledOutcomeLocked` branch (card 5) | D8, D9 |
| `goclient.go`, `slackbridge.go`, `slackcli/wiring.go`, `cli.go` `Bridge:` wiring (card 6) | D3 seam and relay verbs |
| `bridgemessage.go` envelope and `BridgeChannelName` (card 7) | D5, D4 |
| `peers.go`, `bridgemap.go`, `bridgerecords.go`, `geometry.go`, `hubgeom/slackgeom.go`, `api.go` calls (card 8) | D4, D6, D7, D8, D11 |
| `peercommand.go`, `inbound.go` `approve`, `revoke` and `status` app id (card 9) | D6 |
| `bridgeout.go`, `bridgepoll.go`, `relay.go` poll and `settleAccepted` (card 10) | D7, D3, D10 |
| `bridgein.go`, `holder.go` bridge routing and resync hint, `inbound.go` bridge hints (card 11) | D8, D10 |
| `bridgechannel.go`, `keeper.go` operator invites and bridge retire (cards 10 and 12) | D4 |
| `creelhooks.go`, `orch-role.md`, `role-slack.md`, `parent-directive-parent.md` | D9 |
| `docs/slack-app.md` | D11 |
| `PATTERN.md` and `PATTERN-agent-addressing.md` | the declared PATTERN change |
| `PATTERN-sole-parsers.md`, `PATTERN-daemon-wakeups.md`, `cmd/lym/daemonwakeups_test.go` | forced by the record's Constraints (sole parsers, daemon wakeups) |
| tests, `export_test.go`, `testkit/creelfake/bridge.go` | forced by the code they test |

## Per decision

Every decision has builders:

- D1: card 1.
- D2: cards 3 and 5.
- D3: cards 2, 3, 4 and 6, plus `relay.go`.
- D4: cards 7, 10 and 12.
- D5: card 7.
- D6: cards 8 and 9.
- D7: card 10.
- D8: cards 2, 5 and 11.
- D9: card 5 and the contracts.
- D10: `bridgepoll.go` and the `relay.go` wiring.
- D11: `docs/slack-app.md`.
- Arbiter entry: the hour loop, the journal form, the notice record, and the cursor move on approval.

The D8 sweep clause ("the removed-holder sweep journals machine-inbound mail undelivered") has no diff hunk. `sweep.go:239` tests `msg.SendingHub != ""` only. I found no hunk for it and did not trace it further, so I raise no scope finding. It is a correctness question for the code-correctness lens.

## Findings

1. **LOW, class `scope`, `internal/creelcli/cli.go` (the `postCmd` `Long` help text, diff hunk around line 377).**
   - The diff rewrites existing help prose that no decision asks to change. It drops "a hand-added one or a session launched without those hooks".
   - It also drops "and the holder's reply clears it" from the `--expect-reply` text, and rewords "with the id after the #".
   - Only the added "A receiver may be an orch of another machine…" paragraph traces to D2.
   - Evidence: the `cli.go` diff, the hunk starting `@@ -372,20 +377,18 @@`.

No other change is unrequested, and every decision is built.

## Tool calls
1. `grep` of `SendingHub` and `SendingMachine` in `creelengine` `sweep.go`, `receive.go`, `chase.go` and `verbs.go`: the sweep hunk was missing from the diff stat.
2. `Read` of `internal/creelengine/sweep.go` lines 215–265: to see the sweep's handling of mail with a sending hub, which the context lacked.
