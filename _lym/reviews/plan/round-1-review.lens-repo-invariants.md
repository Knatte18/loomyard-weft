```yaml
lens: repo-invariants
outcome: reported
```

**Per-card invariants (repo-invariants lens)**

- **Card 1.** Seam: no `//lym:seam` test in `internal/agentname` (held). Tier: `TestParseMachineAddress` is tier0 (held). Doc: the package comment is extended (held). Order: `ParseMachineAddress` has no production caller until card 3, so it is unreachable from a `main` in between (see F1).
- **Card 2.** Seam: no seam test over `record`, `Message` or `LogEntry` (held). Doc: `internal/creelengine/doc.go` is extended (held). Order: the fold branches are reached through `apply` (held).
- **Card 3.** Seam: none (held). Doc: the `PATTERN.md` line, the `pattern/PATTERN-agent-addressing.md` rules and `doc.go` are listed (held). `internal/creelcli/doc.go:47` says `receiver_live` is left out "for a program or another hub". A machine receiver now also leaves it out. The sentence is incomplete rather than false, and card 6 extends that doc (held, NIT at most). Order: held.
- **Card 4.** Seam: none (held). Doc: `PATTERN-daemon-wakeups` and `doc.go` are extended, and `unboundedWaitFiles` is edited (held). Order: `IsDaemonRefusal` is exported with no production caller until card 10 (see F1).
- **Card 5.** Seam: none (held). Doc: the stamp passages, `CreelTrustSentence` and `role-slack.md` are listed. `internal/shuttleengine/doc.go` holds no trust sentence (held). Order: held.
- **Card 6.** Seam: none (held). Tier: `TestSlackBridge` is tier0 and `TestCreelCLI` is tier1. The card pins no file, so this holds as long as `TestSlackBridge` does not go in tier1's `cli_test.go`. Doc: `creelcli/doc.go` is extended (held). Order: the five `GoClient` machine methods have no production caller until card 10, and the card's ImpactSummary says so (F1).
- **Card 7.** Tier: `TestBridgeMessage` is tier0 (held). Doc: `PATTERN-sole-parsers` and `slackengine/doc.go` are extended (held). Order: `FormatBridgeMessage`, `ParseBridgeMessage` and `BridgeChannelName` have no production caller until cards 10 and 11 (F1).
- **Card 8.** Seam: none in `slackengine` or `hubgeom` (held). The `SlackClient` and `Hub` implementers and literals are covered. Doc: held. Order: `ReadPeers`, `UpdatePeers`, `ReadBridgeMap`, `UpdateBridgeMap`, `ChannelOf` and the new `API` methods have no caller until cards 9 to 11 (F1).
- **Card 9.** The `Hub` literals at `slackcli/wiring.go:330`, `inbound_test.go:636` and `relay_test.go:119` are all listed (held). Doc: `slackengine/doc.go:55` stays true when extended (held).
- **Card 10.** Doc: `internal/slackengine/doc.go:79` reads "It runs the socket holder, the outbound read and a wake loop." The card adds a fourth loop, the bridge poll, but lists `doc.go` only under `Extends`, which only adds text, and says nothing about fixing that line. This is broken (F2). `PATTERN-daemon-wakeups` is extended (held). `Mailbox` implementers are covered (held). Order: card 10 relies on card 12's `inviteOperators` to retry a refused bridge operator invite, but its own rows do not test the retry, so its gate passes (held).
- **Card 11.** Doc: `doc.go` gains the receiving half. Lines 52 and 75 stay true (held). Order: held.
- **Card 12.** Held.

**Findings**

- **F1. MEDIUM, design. Cards 4, 6, 7 and 8, which create production functions the relay calls only in cards 9 to 11.**
  - The functions with no caller reachable from a `main` until a later card:
    - `GoClient.TakeMachine`, `ReportMachine`, `SettleMachineDelivered`, `SettleMachineReturned` and `DeliverMachine` (card 6)
    - `IsDaemonRefusal` (card 4)
    - `FormatBridgeMessage`, `ParseBridgeMessage` and `BridgeChannelName` (card 7)
    - `ReadPeers`, `UpdatePeers`, `ReadBridgeMap`, `UpdateBridgeMap`, `BridgeMap.ChannelOf`, `API.BotUsersNamed`, `MemberChannels`, `PostTimestamped`, `PostThreadReply` and `UserAppID` (card 8)
  - Why this breaks a gate: `PATTERN-test-only-code`'s guard is `cmd/lym/testonlycode_linux_test.go:79–81`, a tier1 `//lym:guard` test. The batch gate runs `[0, 1]` and runs guard tests by name, so any batch that ends after card 4, 6, 7 or 8 and before the consuming card fails that gate. No card plans an allowlist entry and its later removal, and no card co-locates each function with its first caller.
  - Evidence: `cmd/lym/testonlycode_linux_test.go:79–81`, `pattern/PATTERN-test-only-code.md:7–8`, and the ImpactSummaries of card 6 ("no caller until card 10") and card 7 ("nothing calls them until cards 10 and 11").
  - Fix: move each function onto the card that first calls it, or reorder so the creator and its caller share a card.
- **F2. LOW, consistency. Card 10, `internal/slackengine/doc.go`.**
  - Line 79 says the relay runs the socket holder, the outbound read and a wake loop. Card 10 adds the bridge poll as a fourth loop, but lists `doc.go` only under `Extends` and does not rewrite the falsified line.
  - Evidence: `internal/slackengine/doc.go:79`, and card 10's Extends list and Intent ("doc.go gains the sending half").
  - Fix: move `internal/slackengine/doc.go` to card 10's `Edit` and correct the sentence.

**Tool calls**

1. Grepped for `//lym:seam` markers in the touched packages, and for the passages in the `slackcli`, `hubgeom`, `shuttleengine` and `creelcli` docs and the `agentname` package comment that the cards affect. The manifest held no seam-marker search and no doc text beyond the symbol definitions.
2. Read the dead-code guard's tier and guard marker, the `//lym:guard` gate rule, and the `Mailbox`/`SlackClient` implementers and `Hub` literals. The manifest lacked the guard test's tier and the interface implementer list.
3. Grepped the `slackengine` and `creelengine` `doc.go` files, `docs/slack-app.md` and `contracts/orchrole` for statements the cards falsify. The manifest did not include those doc texts.
