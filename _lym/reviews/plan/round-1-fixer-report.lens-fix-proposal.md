```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out, by building a case where the proposed change goes wrong.

**MEDIUM — F2, card 12 (outbound-routing):** the new sentence "The stale line (`reportStale`) and the dropped-message line (`refused`) post through `postTo`'s recovery, and a refusal of either counts toward no message's `refusalCap`." asks for a step `postTo` cannot carry out as signed.
- Card 12's Create line declares `postTo(ctx, record *outboxRecord, message creelengine.Message, orch, text string, sender Sender) error`. That takes a creel message, and its refusal counts "through `refused`".
- A stale line has no message, and the dropped line is posted from inside `refused` itself.
- So under that signature, a refused line either recurses into `refused` or needs a dummy message and a sender the card never names. Neither the card nor the proposal changes `postTo`'s parameters or names who the line posts as.
- Evidence: card 12 Create `outbound.postTo`. Its Intent says "Each post Slack refuses with a code counts once toward `refusalCap` through `refused`".

**MEDIUM — F5, card 9 (socket-holder):** the new row "a route whose `LiveHubs` seam blocks until the row releases it: … every hub of the last live-hub read that stood is owed a resync hint written at the next ping" cannot pass as written.
- The card says "Each ping also reads the live-hub list again and pays every owed resync hint (`payOwed`)". It also gives the serving loop's `readLive(ctx)`, which reads the same `LiveHubs` seam, no bound.
- While the seam still blocks, the next ping's live-hub read blocks the serving loop as well, so the owed hint is never written at that ping.
- The row has to say the seam blocks only the route's call, or that the row releases it before the next ping. Otherwise `readLive` needs its own bound.
- The reviewer's fix asked only that the hint be owed; the fixer added "written at the next ping".

**LOW — F1, card 5 (credentials-record):** a real operator cannot start a relay with card 5 alone.
- Card 5's `RequireConfigured` and `starter.ensure` refuse a machine missing the app-level token.
- But its `login` "reads and writes the bot token alone", so no production path in card 5 writes an `app_token`.
- After card 5 lands and before card 6, `lym slack start` and `away` refuse on every machine, and `EnsureRelay` answers `OutcomeNotConfigured`. Card 5 never states this as an intended in-between state.

I checked the F3, F4 and F6–F9 proposals and found nothing to raise. `RelayDeps{` is built only in `internal/slackcli/wiring.go:243` and `relay_test.go:95`, and both set the seam, so F4's panic guard is safe. `OpenConnection` and `PostAs` are planned symbols (card 1), so F7's doc names exist.

Tool calls:
- One grep, because my context held neither the callers of `RelayDeps{` (for F4's panic), nor whether F7's `OpenConnection` and `PostAs` are planned symbols, nor card 9's `readLive` and `LiveHubs` lines after the renumbering (for F5).
