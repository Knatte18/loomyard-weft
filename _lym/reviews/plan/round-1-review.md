---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/05-two-token-login.md:1
    summary: Card 5 bundles the engine's cross-package credential record with the two-token login CLI, the manifest doc and the PATTERN edits, two independently reviewable surfaces in one card
    fix: Run `lym loom insert-card` to put a new card before card 5 (slug credentials-record) that carries the engine credential record alone. That card Creates Credentials, ReadCredentials, WriteCredentials and Credentials.Missing. It Deletes ReadToken, WriteToken and tokenRecord. It Edits RequireConfigured with its re-sign arrow, starter.ensure, RunRelay, TokenFileName and internal/slackcli#runRelay. It also Edits slackCLI.login and slackCLI.setup, which still read and write only the bot token through Credentials, plus slackCLI.start and slackCLI.away. Its test edits are TestCredentialFiles, slackengine newHarness, TestRunRelay, TestEnsureRelay, slackcli harness.configure, the TestSlackCLI rows that call WriteToken or ReadToken, and TestHubDobbyDeps_PrimeTickEnsuresRelay, with the RequireConfigured, TestCredentialFiles and TestEnsureRelay rows card 5 now plans. Card 5, renumbered 6, keeps the rest. That is LiveHubs, readTokens, fileTokens, liveHubGeometries and fakeLoginSocket; the login, loginCmd, tokenPrompt, tokenRefusal, botScopes, noTokenMessage, slackChecks, wireProduction and slackCLI seam edits; the login rows; docs/slack-app.md; and both PATTERN files.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/11-outbound-routing.md:63
    summary: D8's not_in_channel, is_archived and channel_not_found recovery covers only message posts, while the stale line, the dropped-message line and the inbound answers post with none, so a stale line to a prime channel the bot left stays counted and fails at every read
    fix: In card 11's Intent, add that the stale line (`reportStale`) and the dropped-message line (`refused`) post through `postTo`'s recovery, counting toward no message's refusalCap. Add a `TestOutboundRead` row in which the stale line's post to C1 is refused once with `not_in_channel`, the bot joins C1 and the line posts with its count cleared. In card 10's Intent, add that `inbound.answer` refused with `not_in_channel` joins through the keeper's `join` and posts once more. Add a `TestInboundRead` row for that.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/05-two-token-login.md:105
    summary: Card 5 rewords botScopes and drops groups:history, but its test rows do not update the TestSlackCLI row MissingScopeNamesTheScopes, which asserts groups:history through scopeRefusal, so card 5's own gate fails on it
    fix: In card 5's Intent test paragraph, add that the `MissingScopeNamesTheScopes` row of `TestSlackCLI` now expects the manifest's bot scopes (`channels:history`, `channels:join`, `channels:manage`, `channels:read`, `chat:write`, `chat:write.customize`, `users:read`) and `docs/slack-app.md`, in place of `groups:history`. In card 6's setup rows, keep only that row's switch from a `/conversations.info` to a `/users.info` `missing_scope` answer.
  - id: F4
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-channel-keeper.md:64
    summary: The nil-LiveHubs fallback in card 9's heldElsewhere and card 10's RelayDeps.LiveHubs guards a state no entry point produces, since runRelay and every planned harness set the seam
    fix: In card 9's Intent, replace "A nil `LiveHubs` lists the keeper's own hub alone; the relay's wiring always sets it." with "`newChannelKeeper` panics on a nil `LiveHubs`, a wiring guard; the relay's wiring always sets it." In card 10's Intent, drop ", a nil value listing the relay's own hub alone, as card 9's keeper reads it", and add that card 10's inbound `newHarness` builds its keeper with a `LiveHubs` answering the harness hub.
  - id: F5
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-socket-holder.md:74
    summary: Three prescribed holder branches have no TestSocketHolder row - a route that stalls before it finds any hub, the refused-dial episode ending after a connection serves a minute, and the hint-write episode ending once a write stands
    fix: Add three rows to card 8's TestSocketHolder list. First, a `LiveHubs` seam that blocks until the row releases it, with the clock advanced ten seconds, acks the envelope and owes a resync hint to every hub of the last live-hub read that stood. Second, after the `*APIError` dial notice, a connection that serves a minute, followed by another refused dial, mails a second notice. Third, after the hint-write notice, a write that stands, followed by another failure, mails a second notice.
  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-outbound-routing.md:69
    summary: Card 11 rewords outbound.read's missing-setup refusal to name lym slack setup --operator, but no TestOutboundRead row reaches it, while card 10 plans a reaching row for its inbound counterpart
    fix: Add a row to card 11's `TestOutboundRead` list in which a read with `setup.json` removed returns an error naming the setup path and `lym slack setup --operator <user-id>`, and posts nothing.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/12-relay-push-loop.md:86
    summary: Card 12's doc.go prescription names only themes, not the sentences earlier cards falsify (WriteToken and ReadToken, the inbound cursor, the bot-token bearer header, PostMessage, the inbound poll and its five-second floor)
    fix: In card 12's Docs paragraph, add that `internal/slackengine/doc.go` drops its sentences on `WriteToken` and `ReadToken`, the inbound cursor, the bearer header as the bot token alone, `PostMessage` and the inbound poll with its five-second floor. In their place it names `ReadCredentials` and `WriteCredentials`, the channel map, `PostAs`, and the app-level token for `OpenConnection`. Also add that `internal/slackcli/doc.go` drops `setup` recording the hub's channel.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/11-outbound-routing.md:66
    summary: Card 11's Intent still says a refused reopen, join or ensure adds no second count, though the operator-added entry removes every reopen
    fix: In card 11's Intent, replace "a refused reopen, join or ensure after it adds no second count" with "a refused join or ensure after it adds no second count".
  - id: F9
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/06-setup-operator-status.md:47
    summary: Card 6 says card 12 stamps LastSlackOK at each Socket Mode pong and at the end of each outbound read, but card 7 stamps the outbound read and card 8 the pong
    fix: In card 6's Intent, replace "card 12 stamps it at each Socket Mode pong and at the end of each outbound read" with "card 7 stamps it at the end of each outbound read and card 8 at each Socket Mode pong".
---

### [MEDIUM] Card 5 bundles two independently reviewable surfaces

**Location:** `_lym/plan/05-two-token-login.md:1`

**Issue:** The rubric's granularity rule says a public symbol, or one reused across packages, gets its own card and is not bundled into another surface's card.
Card 5 holds two such surfaces.
The first is the engine's credential record and its refusal: `Credentials`, `ReadCredentials`, `WriteCredentials`, `Missing` and the `RequireConfigured` re-sign.
`internal/slackcli` and `internal/beamcli` read it, and later cards build on it.
The second is the CLI's two-token login: `readTokens`, `fileTokens`, the dial check, the relay-ending login and `liveHubGeometries`.
The card adds `docs/slack-app.md` and the PATTERN background rewrite on top, which makes it the plan's widest card at once.
The record can land first on its own, with login writing only `BotToken` through it, and the login change then reviews by itself.

### [LOW] D8's refusal recovery skips the stale line, the dropped line and the answers

**Location:** `_lym/plan/11-outbound-routing.md:63`

**Issue:** `postTo` recovers once from `is_archived`, `not_in_channel` and `channel_not_found`, but only for a message post.
D8 says a post refused with `not_in_channel` brings the bot back and posts again, and the relay's stale and dropped lines are posts.
Today a stale line that fails keeps its count (`internal/slackengine/outbound.go:136-147`).
So with the bot out of the prime's channel, the line fails at every read until a message post happens to rejoin the bot.
Card 10's `answer` also posts with no recovery (`10-inbound-push.md:102`).
The fork rated this MEDIUM; it is kept at LOW because the gap closes at the next message post to that channel.

### [LOW] Card 5's botScopes change breaks a row only card 6 rewrites

**Location:** `_lym/plan/05-two-token-login.md:105`

**Issue:** `scopeRefusal` formats `botScopes` (`internal/slackcli/cli.go:360`).
The `MissingScopeNamesTheScopes` row asserts `groups:history` in that refusal (`internal/slackcli/cli_test.go:481`).
Card 5 drops that scope, and `TestSlackCLI` is among its targets.
Its test paragraph names no update to the row, though, and card 6 is the first card that rewrites it, so card 5 fails its own gate as planned.

### [LOW] The nil-LiveHubs fallback guards an impossible state

**Location:** `_lym/plan/09-channel-keeper.md:64`

**Issue:** Card 9 says a nil `LiveHubs` lists the keeper's own hub alone and that the relay's wiring always sets it.
Card 10 repeats this for `RelayDeps.LiveHubs` (`10-inbound-push.md:109`).
Production `runRelay`, `newRelayRun`, `newKeeperHarness` and `newOutboundHarness` all set it, so no input reaches the branch.
The plan-writer stencil says a branch that upstream wiring rules out is dropped or written as a fail-loud wiring guard.

### [LOW] Card 8 leaves three prescribed holder branches without a row

**Location:** `_lym/plan/08-socket-holder.md:74`

**Issue:** Three branches have no row.
- **Route stalled before finding a hub:** card 8 prescribes a resync owed "to every hub of the last live-hub read that stood when it found none yet". The only stall row blocks the hint writer after the route has found B (`:100`).
- **Refused-dial episode end:** the episode ends once a connection serves a minute (`:63`). No row asserts a second notice after that.
- **Hint-write episode end:** the same holds once a hint write stands again (`:84`).
Only the `LiveHubs` episode's end has a row.

### [LOW] Card 11's reworded missing-setup refusal has no reaching row

**Location:** `_lym/plan/11-outbound-routing.md:69`

**Issue:** `read`'s missing-setup error is reworded to name `lym slack setup --operator <user-id>`.
No `TestOutboundRead` row today or in card 11 removes `setup.json`.
The rubric's reachability bullet needs a named test that reaches a prescribed refusal.

### [LOW] Card 12's doc.go prescription does not name the stale sentences

**Location:** `_lym/plan/12-relay-push-loop.md:86`

**Issue:** `internal/slackengine/doc.go` names `WriteToken` and `ReadToken` (line 8), the inbound cursor (line 11), the bot token as the bearer header (line 26), `PostMessage` (line 29) and the inbound poll with its five-second floor (lines 37 and 56–57).
Cards 1, 5, 7 and 10 falsify those sentences.
`doc-go-unlisted` passes on card 12's Edit, but its Docs paragraph names only the themes to describe, so stale symbol names can survive the rewrite.

### [LOW] Card 11 keeps the word "reopen" from before the operator's override

**Location:** `_lym/plan/11-outbound-routing.md:66`

**Issue:** The decision record's operator-added entry says lym never reopens an archived channel; `is_archived` marks the entry archived and ensures a new channel.
Card 11's own recovery list follows that, but the count sentence still names "a refused reopen".

### [NIT] Card 6 credits the last-healthy stamps to the wrong card

**Location:** `_lym/plan/06-setup-operator-status.md:47`

**Issue:** Card 7 moves the stamp to the end of each outbound read, and card 8's holder stamps at each pong.
Card 6 credits both to card 12, and the `RelayState.LastSlackOK` comment it describes is rewritten from that text.

## Focus verdicts

- **Card 5 Delete or Edit:** not a defect.
  `pattern/PATTERN-agent-addressing.md` and `PATTERN.md` sit under card 5's `**Edit:**`, not `**Delete:**`; the Delete group holds `ReadToken`, `WriteToken`, `tokenRecord` and `readToken`.
  The background's rule is rewritten, so it belongs under Edit.
  The format block requires a declared `changes` entry to be fenced as its whole overview line on a card that edits `PATTERN.md`.
  The fenced line matches `PATTERN.md:43` verbatim, as the record's Constraint asks.
- **Card 4 Extends on `internal/hubgeom/doc.go`:** not a defect.
  The `SlackGeometry` paragraph (`doc.go:23-26`) names no cursor or other field the card removes.
  The card adds the machine name and the socket lock beside the token sentence and rewords nothing.
- **Re-sign arrows on cards 5, 10, 11 and 12:** not a defect for any listed member.
  - `starter.ensure`, `outbound.read` and `runInbound` keep their signatures in the cards' Intents.
  - `runOutbound`, `handle`, `NewInbound`, the outbound constructors and `RequireConfigured` carry arrows.
  - `readFailed`'s per-orch mark is in the Rejected list below.
  - Symbol-naming Intents are carried by member glyphs. The bare file targets are non-Go files, `doc.go` files or new files.
- **Refusal reach:** the interrupted read and the empty line in card 5 are reached by the existing `terminalRefusals` and `EmptyStandardInputIsRefusedWithoutAskingSlack` rows (`internal/slackcli/cli_test.go:405-446`). Those rows stay in `TestSlackCLI`. The gaps found are F4, F5 and F6.
- **Stale pre-override wording:** only F8. No card keeps the user token, `conversations.unarchive` or `stop` archiving.
- **HelloTimeout, `then lym slack start` and `role-slack.md`:** all licensed.
  - The hello bound sits inside D11's client and bounds D3's and D1's hello read.
  - The added way forward follows the `PATTERN-refusal-way-forward` Constraint that each new relay-start refusal names its way forward.
  - `role-slack.md:22-23` describes the `@<slug>` answer hint D7 removes, so D7 forces its edit.
- **Fidelity sweep:** every Decision and Constraint has a carrying card.
  - D3's relay-ending login is card 5.
  - D5's per-orch lock is card 9, and its triggers are cards 11 and 12.
  - D2's 24-hour scan and prune floor are cards 4 and 10.
  - D8's stale line to the prime is card 11.
  - Every `lym slack setup --channel` site is reworded by cards 5, 6, 10 and 11, the sites the affected-set lens enumerated.
  - Every way forward that lists bot scopes names `docs/slack-app.md`, through `botScopes` in card 5 and `readFailed` in card 10.
- **Granularity:** F1 for card 5.
  - Card 10's cursor-geometry removal is forced by the deleted cursor, and its test-harness reshaping serves that one surface.
  - Card 12's docs follow the stencil's rule that the code's card writes its docs.
  - The private helpers listed under Create follow the format's rule that every new member is a declared handle.
- **Test economy:** no finding.
  - `ArchivedName` rides rows of the one naming table, `TestChannelName`, with no test of its own.
  - Card 12's two-hub row exercises `holderRun`'s wiring and B's forward, which card 8's holder rows cannot reach.
  - The card 5 and card 6 rows assert refusal behaviour, not static help text.
- **Card order:** held.
  - Card 5's `RunRelay` and `starter.ensure` edits precede card 7's.
  - Card 6's `ConversationInfo` delete follows card 1's edit of its test expectation.
  - `RelayDeps.LiveHubs` and the shared keeper come in card 10, before cards 11 and 12 use them.
  - No card 5 or card 6 target calls `API.PostMessage`.
- **ImpactSummary and attack surface:** held.
  - Card 12's summary states its blast radius (`runRelay`, the doc files, one stop assertion) rather than restating its Intent.
  - Card 8's hint writer states that a hint carries no text.
  - Card 9's join and adopt state the live-hub-list bound.
  - Card 5's relay-ending login states the machine-record bound.
- **Card 8's temp file:** the overview records it as a `PATTERN-lym-files` departure. It is the same write-then-rename staging that `internal/state`'s atomic writes use, and D1 mandates it, so it raises no BLOCKING finding.

## Rejected

- **`liveHubGeometries`' per-hub geometry failure unreachable** (guard-reach F2): rejected.
  That branch returns the error of a `SlackGeometry` call, which Go must handle anyway; the card prescribes only its wording, not a guard.
  The `hubregistry.Live` failure is a propagated read or decode error, which the rubric exempts.
- **Card 11's `read` guards a file the outbound no longer reads** (the second half of guard-reach F6): rejected.
  The keeper the outbound ensures through reads `setup.json` for each operator invite, so the file still matters there. Only the missing reaching row is kept, as F6.
- **A missing re-sign arrow on `inbound.readFailed`** (handler candidate from the focus): rejected.
  Card 10's Intent changes the notice wording and keys the episode mark per orch, but it states no change to the method's parameters or results.
  The rubric bullet fires only when the Intent changes them, and every caller of `readFailed` after card 10 is a method that card itself creates.
