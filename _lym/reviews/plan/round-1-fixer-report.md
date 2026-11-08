# Fixer report — plan, round 1

All nine findings are fixed; one is fixed with the fixer's own change, and the others are fixed as proposed.
Card numbers below are the numbering after F1's insert: the old cards 5 to 12 are now 6 to 13.
`lym loom validate-plan` answers `ok: true`, with one informational finding for the new unit `docs/slack-app.md`.

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  `lym loom insert-card 5 credentials-record` added `05-credentials-record.md` and renumbered the later cards and their references.
  Card 5 carries the credential record:
  - Create: `Credentials`, `ReadCredentials`, `WriteCredentials` and `Credentials.Missing`.
  - Delete: `ReadToken`, `WriteToken` and `tokenRecord`.
  - Edit: `RequireConfigured` with its re-sign arrow, `starter.ensure`, `RunRelay`, `TokenFileName`, `internal/slackcli#runRelay`, and `login`, `setup`, `start` and `away` over the record.
  - Tests: `TestCredentialFiles`, `TestEnsureRelay`, slackengine `newHarness`, `TestRunRelay`, slackcli `configure`, the `TestSlackCLI` rows that call `WriteToken` or `ReadToken` with `StartAndAwayNeedLoginAndSetup`, and `TestHubDobbyDeps_PrimeTickEnsuresRelay`.
  Until card 6, login replaces only `BotToken` and keeps a stored app-level token and app id; the `StoresTheTokenOnlyAfterSlackAcceptsIt` row asserts that.
  Card 6, two-token-login, keeps the login surface, `LiveHubs`, `docs/slack-app.md` and both PATTERN files.
  It uses the record and names card 5 as its source, and its ImpactSummary and its Card Index line are narrowed to the login.
  The overview's opening paragraph names the credential record.
- **F2 (LOW), fixed differently.**
  Card 12 sends the stale line and the dropped-message line through the recovery, counting toward no message's `refusalCap`, and adds the `TestOutboundRead` row for a stale line refused with `not_in_channel`.
  Card 11 adds that an `answer` refused with `not_in_channel` joins through the keeper's `join`, with its `TestInboundRead` row.
  Not made as proposed: posting the lines "through `postTo`'s recovery".
  `postTo` takes a creel message and counts each refusal through `refused`, and `refused` itself posts the dropped line, so the proposed wording describes a loop and a count with no message (self-check, fix-consistency and fix-proposal).
  Card 12 instead creates `outbound.postRecovered(ctx, orch, text, sender)`, which holds the three recoveries.
  `postTo` posts a message through it and counts, while the two lines call it directly under `<shortname>:SLACK` with `programIcon`.
- **F3 (LOW), fixed as proposed.**
  Card 6's test paragraph says `MissingScopeNamesTheScopes` now expects the manifest's bot scopes and `docs/slack-app.md` in place of `groups:history`.
  Card 7's setup row keeps only the switch from `/conversations.info` to `/users.info`.
- **F4 (LOW), fixed as proposed.**
  Card 10: `newChannelKeeper` panics on a nil `LiveHubs`, a wiring guard.
  Card 11 drops the nil fallback and says its inbound `newHarness` builds its keeper with a `LiveHubs` answering the harness hub.
- **F5 (LOW), fixed as proposed.**
  Card 9 adds three `TestSocketHolder` rows: a route whose `LiveHubs` seam blocks, a refused-dial episode ending after a minute-long connection, and a hint-write episode ending once a write stands.
  The self-check found that the blocked seam would also block the next ping's `readLive`, so the row releases the seam before that ping.
- **F6 (LOW), fixed as proposed.**
  Card 12 adds a `TestOutboundRead` row: with `setup.json` removed, a read returns an error naming the setup path and `lym slack setup --operator <user-id>`, and posts nothing.
- **F7 (LOW), fixed as proposed.**
  Card 13's Docs paragraph names the `internal/slackengine/doc.go` sentences that go and what replaces them, and says `internal/slackcli/doc.go` drops `setup` recording the channel.
  The self-check found further stale sentences, and the paragraph now covers them too: `slackengine/doc.go`'s setup file with the channel and its posting "to the channel", and `slackcli/doc.go`'s one-token login sentences.
- **F8 (LOW), fixed as proposed.**
  Card 12: "a refused join or ensure after it adds no second count".
- **F9 (NIT), fixed as proposed.**
  Card 7 says card 8 stamps `LastSlackOK` at the end of each outbound read and card 9 at each Socket Mode pong. The text was written as "card 7" and "card 8" and renumbered by the insert.

## Self-check

The forks found five problems, and two of their findings describe the same one.
- fix-consistency, MEDIUM: the new lines-through-recovery sentence contradicts `postTo`'s signature and its counting through `refused`. Fixed with own change, merged with the fix-proposal F2 finding: the new `postRecovered`, under F2 above.
- fix-proposal, MEDIUM: F2's proposal cannot be carried out under `postTo` as signed. Same finding as above, fixed with own change.
- fix-proposal, MEDIUM: F5's blocked-`LiveHubs` row also blocks the next ping's live-hub read. Fixed as proposed: the row releases the seam before the ping.
- fix-proposal, LOW: card 5 alone leaves no production path that writes an app-level token. Fixed as proposed: card 5 states this in-between state until card 6's login.
- fix-affected-set, LOW: F7 missed `internal/slackengine/doc.go:10` and `:44`. Fixed as proposed in card 13's Docs paragraph.
- fix-affected-set, LOW: F7 missed `internal/slackcli/doc.go`'s one-token login sentences. Fixed as proposed in card 13's Docs paragraph.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
