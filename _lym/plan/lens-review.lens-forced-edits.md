```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: the edits the plan's changes force outside the members its cards list.

1. **MEDIUM — D9 and the machine-mail fields; cards 2, 5 and 6: the creel CLI's JSON writers never show machine mail's trust, sending machine or Slack timestamp.**
   - `internal/creelcli#messageFields` (cli.go:302-330) emits `trust`, `sending_id` and `sending_repository` only under `if message.SendingHub != ""`.
   - A machine-inbound message has `SendingMachine` set and `SendingHub` empty, so `lym creel read --json` drops its trust and its sending id.
   - `internal/creelcli#logEntryFields` (cli.go:836-878, the map at 865-866) lists `sending_hub`, `sending_shortname` and `receiving_hub` by name. The `ReceivingMachine`, `SendingMachine` and `SlackTS` that card 2 adds to `LogEntry` are therefore never printed.
   - `internal/creelcli/doc.go:27` and `:32` describe those field sets.
   - No card lists `messageFields`, `trustField` or `logEntryFields`. Card 6 only extends `doc.go` with the verbs.
   - Fix: on card 6, add `internal/creelcli#messageFields` and `internal/creelcli#logEntryFields` under Edit, with rows in `TestCreelCLI`, and say in the doc.go text that the field lists change.

2. **MEDIUM — card 10, `RunRelay` and the bridge poll: the bridge poll's waits and calls land in `TestRunRelay`, which no card lists.**
   - Card 10 adds to `RunRelay` a goroutine that waits on `deps.Clock` from 5s up to 60s. It also adds `settleAccepted` and a `TakeMachine` call on every poll.
   - `TestRunRelay`'s harness runs on `manualClock` and keys its asserts on the waits it sees (`awaitWait(t, d)` and `hasWait(d)`, relay_test.go:46). The outbound floor it waits for is also 5s, so the new 5s bridge wait makes those keys ambiguous.
   - Its harness also counts mailbox calls (`readAllCount`, relay_test.go:39).
   - Neither `internal/slackengine_test#TestRunRelay` nor `internal/slackengine_test#newRelayRun` appears on card 10 or card 11.
   - Fix: add both under card 10's Edit. State that the harness tells the bridge poll's waits apart, and that the scripted take queue is empty.

3. **MEDIUM — cards 9 and 11, `Hub.MachineNames`: the third `slackengine.Hub` literal is never updated.**
   - Card 9 updates two `Hub` builders: `runRelay` and `fakeHub` (inbound_test.go:636).
   - The third, in `internal/slackengine_test#newRelayRun` (relay_test.go:119), is listed nowhere. It leaves `MachineNames` nil.
   - Card 11's reader calls `Hub.MachineNames` for every envelope (step 6) and during resync. A relay run in `TestRunRelay` therefore calls a nil func, and no card says what a nil `MachineNames` means.
   - Fix: add `newRelayRun` to card 9 or card 11, or state a meaning for a nil field.

4. **LOW — card 9, the reworded `status` answer: existing exact assertions must change, and the Intent does not say so.**
   - The answer changes from `presence is %s` to `presence is %s; this machine's app id is %s`.
   - `internal/slackengine/inbound_test.go:1293` and `:1415` assert `"presence is slack"` and `"presence is terminal"` through `wantAnswers`.
   - `TestInboundRead` is on the card, but the Intent names only new rows. The implementer is not told that these existing rows must be reworded (rule 4 of "What a card is").

5. **LOW — card 3, comments that the new edge falsifies.**
   - `internal/creelengine#PostResult` says `ReceiverLive` is nil "for a program and for a strand of another hub". A machine post leaves it nil too.
   - `internal/creelengine#PostTags` says "a post to another hub drops them". `postMachine` drops them too.
   - `internal/creelcli#postCmd`'s help (cli.go:402) lists another hub's strand as a receiver, but not `<machine>::<name>`.
   - The `forward` help (cli.go:610) names only another hub.
   - No card lists any of these.

6. **LOW — card 1's untargeted edit: the Intent edits the package comment, but no target names its file.**
   - Card 1's Intent adds a sentence to the package comment in `internal/agentname/agentname.go`. The package has no `doc.go`, and its comment sits in that file.
   - The card has only Create and Uses groups, so no target names that file.

7. **LOW — card 8, a misplaced target: `internal/slackengine#ChannelInfo` sits under Edit, yet the Intent says "`ChannelInfo` keeps its fields".**
   - The type is only read. It belongs under Uses, or should be dropped.

Further tool calls:
- A grep for `SendingHub` in non-test creelengine/creelcli code, for `noticeReturnedLocked` callers, for `presence is` answer strings, for `slackengine.Hub{` literals and for `ReadWait` implementers. My context lacked the call sites outside the files I had read.
- A grep of creelengine's import guard and other import allowlists, of the trust-sentence text across the repo, and of the guards on `PATTERN-sole-parsers`, plus a `lym quarry` lookup of creelcli functions. My context lacked whether the new imports and texts hit any guard or fixture.
- A read of `messageFields`/`trustField`, of `logEntryFields` lines 855-872, and of `creelcli/doc.go`'s field lines. My context lacked the bodies of the JSON writers that print machine mail.
