```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch or test row that cards 1–12 add can be reached from a real entry point, and whether a guarded site recurs unguarded elsewhere.

## Per guard, reachable or not

- **Card 1**: `Invite` answering `already_in_channel`, `Archive` answering `already_archived`, and `FindChannel` finding nothing are each reached by a named `TestAPI` row.
- **Card 2**: the dial's `*APIError`, a first message that is no `hello`, and no `hello` within the bound are each reached by rows 2–4 of `TestSocketClient`.
- **Card 4**: `ReadChannelMap`'s decode error is reached by the row with a map holding `{`. The `machinename.Name` error is exempt as a propagated read.
- **Card 5**:
  - `readTokens`, interrupted read and empty line: reachable. The existing `terminalRefusals` rows (`cli_test.go:405-426`) reach both, and the harness's `readHiddenLine` returns `terminalErr` on the first of the two reads.
  - Empty piped input: reachable through the existing `EmptyStandardInputIsRefusedWithoutAskingSlack` row (`cli_test.go:438`). Which refusal it hits is unclear, `readTokens`' empty refusal or `fileTokens`' "not one of each".
  - Every `fileTokens` refusal (`xoxp-`, two bot tokens, a lone bot token), the hello app id `A2`, a refused `auth.test`, `apps.connections.open` or `bots.info`, a transport-failed dial, the relay-ending login and the failed `liveHubs` read after the write: each has a named row.
  - `RequireConfigured`'s four kinds and `starter.ensure` with a bot token alone: reachable through named rows.
  - **`liveHubGeometries`' per-hub geometry failure: not reachable** (finding F2).
- **Card 6**: the `users.info` refusals, `missing_scope`, a missing token, `--channel` as an unknown flag, and a status over a map holding `{` are each reached by a named row.
- **Card 8**:
  - Reachable through named rows: the lock wait that ctx ends, an `*APIError` dial, a `link_disabled` disconnect, a failing hint writer, a hint writer stalled past `envelopeLimit` after the route found B, an unmapped event while the maps read or while one is unreadable, a failing `LiveHubs`, a failed resync after a connection opens, and a ping failing or blocking past `pongTimeout`.
  - **A route that stalls before finding any hub: not reached by any test** (F4).
  - **The two episode ends (a connection serving a minute; a hint write standing again): not reached by any test** (F5).
- **Card 9**:
  - Reachable through named rows: the invite refusal notice, `AuthTest` failing on `name_taken`, a name `FindChannel` no longer finds, an archived or foreign channel on `name_taken`, a channel held by the other hub's map, a map holding `{` or `LiveHubs` failing (fails closed), and the retire branches (`-2` suffix, refused archive, `not_authed` rename, the prime never renamed).
  - The "held by another orch of this hub" branch is reachable, though only by contrivance: a beam slug such as `foo-2026-11-08` makes a D4 name equal to beam `foo`'s dated name.
  - **The nil-`LiveHubs` branch: not reachable** (F3).
- **Card 10**:
  - Reachable through named rows: a hint that does not parse or names an unmapped channel, a `.`-prefixed hint, a drain with no `setup.json`, the per-orch `readFailed` episode, the owed read after a transport error, an HTTP 500 or a rate limit, and `not_in_channel` on a start or resync read.
  - **The nil `RelayDeps.LiveHubs` branch: not reachable** (F3).
- **Card 11**:
  - Reachable through named rows: `outboundRoute`'s refusals (the existing `NonOrchStrandAndExternalMailIsDroppedAndAcknowledged` row), `errNoOpenChannel`, `errChannelRefused` up to the cap, the prime refused too, the `is_archived`, `not_in_channel` and `channel_not_found` recoveries, a refused join counting once, and a transient ensure failure.
  - **`read`'s reworded missing-setup refusal: no named test reaches it** (F6).
  - **The three-refusal recovery recurs unguarded on other posts** (F1).
- **Card 12**:
  - `missingTokenExit`'s three rows are reachable. In production the real input is a re-exec from an older binary, since `starter.ensure` returns `OutcomeNotConfigured` without spawning a relay when a token is missing.
  - Also reachable through named rows: `Watch` failing into `diskCheck`, the one-second wake floor, a file event during an outbound wake, a failed exec restarting the holder, a remembered prime ensure, a refused retire, and the presence change from `terminal` to `slack`.

## Findings

**F1: the three-refusal recovery recurs unguarded on every other post.**
- Severity MEDIUM, class design, at card 11 `postTo` and card 10 `inbound.answer`.
- `postTo`'s recovery from `is_archived`, `not_in_channel` and `channel_not_found` covers only the message posts.
- The stale line, the dropped-message line, the inbound answers and the paused line post with no recovery, and so does card 12's `missingTokenExit` line.
- With the bot out of the prime's channel, the stale line keeps its count and fails at every read until some message post happens to rejoin the bot. Answers in a beam channel the bot left are lost and only logged.
- D8 says "A post refused with `not_in_channel` invites the bot back … and posts again".
- Evidence: `_lym/plan/11-outbound-routing.md:930-935` (the stale line is not routed through `postTo`), `10-inbound-push.md:831`, and `internal/slackengine/outbound.go:136-147` (a stale line's failure keeps the count).

**F2: `liveHubGeometries`' per-hub geometry failure has no reachable state and no test.**
- Severity LOW, class design, at card 5.
- `hubregistry.Live` already keeps only hubs whose Bolt record reads their own shortname (`internal/hubregistry/hubregistry.go:98-107`).
- `SlackGeometry`'s other failures are machine-wide: the credential dir, the executable and the machine name (`internal/hubgeom/slackgeom.go:26-33`). Those fail the caller's own geometry first, in `wireProduction` or `runRelay` (`internal/slackcli/wiring.go:102,231`).
- So the branch is hit only by a race on a shortname read, and no row names it.
- The `Live` failure itself already names the record path through `refuseUnreadable`. No row reaches the production error that card 8's notice quotes, because every test fakes the seam's error.
- Evidence: `_lym/plan/05-two-token-login.md:348-350`.

**F3: the nil-`LiveHubs` fallback guards a state no entry point produces.**
- Severity LOW, class design, at card 9 `heldElsewhere` and card 10 `RelayDeps.LiveHubs`.
- "A nil `LiveHubs` lists the keeper's own hub alone; the relay's wiring always sets it." Card 10 says the same of `RelayDeps`.
- Production `runRelay` always fills the seam, and `newRelayRun` and `newOutboundHarness` set it, so no input and no test reaches the branch.
- Drop it, or make it a fail-loud wiring guard.
- Evidence: `_lym/plan/09-channel-keeper.md:699`, `_lym/plan/10-inbound-push.md:838-839`, `11-outbound-routing.md:939`.

**F4: a route that stalls before finding any hub is not reached by any test.**
- Severity LOW, class design, at card 8 `route` with `envelopeLimit`.
- The prescribed branch is "a resync hint owed … to every hub of the last live-hub read that stood when it found none yet".
- A map read blocked on its lock, or a `LiveHubs` call that hangs, is a real input. The only stall row blocks the hint writer after the route has already found B.
- Evidence: `_lym/plan/08-socket-holder.md:600` against row `:626`.

**F5: the episode-ending branches are not reached by any test.**
- Severity LOW, class design, at card 8's notices.
- "The next connection that serves a minute ends the episode." No row asserts a second notice after a recovered episode.
- The same holds for the hint-write episode's end. Only the `LiveHubs` episode's end is asserted, at `:628`.
- The existing inbound test pins its per-episode reset, at `internal/slackengine/inbound_test.go:889-897`.
- Evidence: `_lym/plan/08-socket-holder.md:589,610,625,632`.

**F6: `outbound.read`'s reworded missing-setup refusal has no reaching test, and guards a file the outbound no longer reads.**
- Severity LOW, class design, at card 11.
- After card 11 the outbound takes no channel from `setup.json`; the signatures drop `setup`. Yet `read` still refuses every post while the file is missing, and no `TestOutboundRead` row reaches that refusal.
- Card 10 has a reaching row for its inbound counterpart (`10-inbound-push.md:848`).
- Evidence: `_lym/plan/11-outbound-routing.md:924,936`, `internal/slackengine/outbound.go:76-82`.

Tool calls:
- `sed -n 90,130p internal/hubregistry/hubregistry.go`: `Live`'s error and filter paths, to judge whether `liveHubGeometries`' per-hub failure state can occur. The manifest had only `Live`'s signature.
