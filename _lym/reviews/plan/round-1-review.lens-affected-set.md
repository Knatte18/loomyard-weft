```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change the cards make, I listed what reads or asserts it and which cards cover that set.

**Per change: the readers found, and the cards that cover them**

- **`CreelTrustSentence` text and the `(other hub, trusted orch)` passages** (card 5).
  - Readers: `internal/shuttleengine/creelhooks.go:17`, which holds the passage `CreelTrustSentence`; `claudeengine/settings.go:547,559`, which checks the constant holds no newline; `codexengine/developerinstructions.go:33`; and the claudeengine and codexengine tests, which compare against the constant.
  - Passages: `contracts/orchrole/orch-role.md:419` and `contracts/stencils/parent/parent-directive-parent.md:18`.
  - `internal/creelengine/doc.go:765` still describes the hub stamps correctly.
  - A repo-wide search found no other passage outside `docs/research` data.
  - Covered by card 5, plus the doc.go Extends of cards 2 to 5.
- **`status` answer wording** (card 9). Readers: `internal/slackengine/inbound_test.go:1293,1366,1415,1534`. Covered by card 9's `TestInboundRead` edit. No doc quotes the answer: `docs/slack-app.md` and `role-slack.md` don't, and `slackengine/doc.go:55` only names the commands.
- **`Hub` gains `MachineNames`** (card 9). Builders: `slackcli/wiring.go:330` (`runRelay`), `inbound_test.go:636` (`fakeHub.hub`), `relay_test.go:119` (`newRelayRun`). Card 9 covers all three.
- **`Geometry` and `MachinePaths` gain paths** (card 8).
  - Covered by card 8: `hubgeom/slackgeom.go:25,54`, the full-struct wants at `hubgeom_test.go:525,582` (`TestSlackGeometry`, `TestSlackMachine`), and `newGeometry` (`inbound_test.go:677`).
  - The partial named literals at `slackcli/cli_test.go:244` and `slackengine/files_test.go:269` compile unchanged and reach no bridge path, so they are not forced.
- **`SlackClient` grows** (card 8).
  - Implementers: `*API` and `fakeSlack`, both covered by card 8.
  - The reflective scan at `keeper_test.go:438` only bars a method named "unarchive", which no new method is.
- **`Mailbox` grows** (card 10). Implementers: `slackcli`'s `programMailbox` (`wiring.go:244`) and `fakeMailbox`; card 10 covers both. `GoClient` is not used as a `Mailbox`.
- **`noticeReturnedLocked` re-sign** (card 4). Its one caller, `settleReturned` (`outbound.go:150`), is on card 4.
- **`SlackProgram` = `BridgeProgram`** (card 4). The value is unchanged; readers in `beamcli/dobbydeps.go:371`, `slackcli/wiring.go:254`, `slackcli/hook.go:188` and `slackengine/outbound.go:401` are unaffected.
- **Guards widened from `SendingHub` to `SendingMachine`**:
  - `Reply` (`verbs.go:331`), `Forward` (`verbs.go:1012`) and `releaseGoneHolder` (`sweep.go:239`): card 5.
  - `messageFields` (`creelcli/cli.go:321`): card 6.
  - `HubStamp` (`fold.go:102`): card 5.
- **New `LogEntry` fields** (card 2). `logEntryFields` (`cli.go:866`) and the JSON field list in `creelcli/doc.go:27,32` are covered by card 6.
- **`Request`/`Response` new fields** (cards 4, 5). The `GoClient` methods are on card 6.
- **`refuseOtherHub` refusing `::` names** (card 3). `inject`, `interrupt`, `hold`, `release` and `post --interrupt` are covered through card 3's `TestStore_InjectAndHold` row. The `forward` help (`cli.go:610`) and `post` help (`cli.go:402`) are on card 3.
- **New unbounded-wait files** (cards 4, 10). `cmd/lym#unboundedWaitFiles` and the `PATTERN-daemon-wakeups.md` loop list are on both cards.
- **Envelope parser** (card 7). The `PATTERN-sole-parsers.md` list is on card 7.
- **Holder routing of unmapped events** (card 11). `holder.go`'s `route` doc comment is on card 11 as an Edit target. `slackengine/doc.go:51` stays true and card 11 extends it. The existing row `UnmappedEventsAreAckedOnlyWhileEveryMapReads` (`holder_test.go:746`) sits in `TestSocketHolder`, which card 11 edits. I did not read its body, so whether it asserts that no hint is written is unverified.

**Findings**

1. NIT · consistency · `internal/creelcli#trustField`. Card 6 has `messageFields` emit `trust` through `trustField` for machine mail, but `trustField`'s doc comment says "the trust value of a message from another hub". Card 6 lists `messageFields`, not `trustField`, so the comment is left incomplete. Evidence: `internal/creelcli/cli.go:332`. Fix: add `internal/creelcli#trustField` under card 6's `**Edit:**` and reword the comment to cover a message from another hub or another machine.

No other affected set was left partly covered.

Tool calls:
1. Repo-wide grep for stamp passages, `presence is`, `Hub`/`Geometry`/`SlackClient`/`Mailbox` literals, `SendingHub` readers, the `trustField` span and docs naming the JSON fields. The manifest had only targeted reads, not repo-wide readers.
2. `keeper_test.go:425-460` (a reflective use of `SlackClient`), `inbound_test.go:630-640` (`Hub` literal), `docs/overview.md`, `docs/slack-app.md` and `role-slack.md` greps, `creelcli/doc.go` field lines. These are readers found in call 1, not in the manifest.
3. `docs/overview.md:905-920`, the `Mailbox` and `SlackClient` implementers, other trust passages, `creelcli` help strings. Needed to confirm the implementer sets and the help text.
4. `docs/slack-app.md` existence and wording, `files_test.go` literals, `role-slack.md` and `slackengine/doc.go` command lines. Needed to confirm which docs describe the commands and the geometry.
5. Grep for `status`-answer assertions in tests, and `doc.go`/`holder_test` drop wording. Call 1's output was truncated by `head`.
6. Grep of `slackengine/doc.go` and `holder_test.go` for unmapped-event wording. Needed to check card 11's routing change against existing docs and tests.
