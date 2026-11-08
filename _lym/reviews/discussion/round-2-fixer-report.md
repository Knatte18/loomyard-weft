# Fixer report — discussion round 2

Every finding's premise held against the record and the code it cites; every finding is fixed.

## Changes

- F1: D2 now removes each hint before the read it triggers, so a hint written during the read is a new wake.
  Added a D2 criterion: a hint written while its channel's read is in flight is read at the next wake.
- F2: on `name_taken`, D5's archived bullet now writes a new entry with its cursor and thread floor at the channel's newest message.
  D6 keeps the cursors only for an entry the map already holds, and D2's thread-floor sentence names the entry-less reopen.
  Added a D5 criterion: a reopen with no map entry forwards no earlier message.
- F3: D1 stamps a last-healthy time, the field `LastSlackOK` holds today.
  It is stamped at each pong for the holder and at each outbound read's end for every relay, and the restart cap and `status` read it.
  The Notes bullet on `EnsureRelayWhileAway` now names this change.
  Added a D1 criterion: an idle relay over ten minutes is no young death.
  Fixed slightly differently: `status` reports the stamp as `last_healthy` in place of `last_slack_ok`, because the stamp now means liveness, not a Slack call that succeeded (the fix-proposal self-check finding).
- F4: D3's missing-token line now posts to `setup.json`'s old channel key when the map has no prime's channel.
  D10 says only that line reads the key, and the D3 criterion covers the re-exec from an older binary's state.
- F5: D5's third ensure bullet now ensures at a relay start while presence is `slack`, and at a wake that reads `slack` after `terminal`.
  The D5 criterion now covers a relay `lym slack away` starts and one restarted while away.
- F6: D1 leaves every event unacknowledged while `hubregistry.Live` fails.
  D3 and D5 fail the map test closed the same way, with one notice naming the hubs record and its way forward.
  The D1 and D3 criteria now cover both.
- F7: D5's map entry records the operator it invited.
  Each ensure of a held channel invites a changed or not-yet-invited operator until the invite stands, and step 3 records it.
  The D5 criterion now says a failed invite is retried at the next ensure and a changed operator is invited.
- F8: D1 writes a hint as a temp file in `.lym/slack/` that is then renamed into the inbox, and D2's drain ignores dot names.
- F9: in D3, a login that changes a stored token ends every live hub's relay.
  D8 and D3's user-token list invite the bot back after a start or resync read refused with `not_in_channel`.
  The rationale now names both invite paths, and D3 and D8 have criteria for these changes.
- F10: D3 refuses an app-level or user token logged in with no bot token stored, naming the bot token first.
  Login stores the app id from the `hello` in `slack.json`, and a D3 criterion covers both.
- F11: in D5 step 2, a live own channel the bot is not a member of has the bot invited first, then is adopted.
  D3's user-token list and criterion name that use, and the D5 criterion drops "the bot belongs to".
- F12: D2's thread read skips the message whose timestamp is the thread timestamp.
- F13: D1 has a holder whose re-exec fails rejoin the machine socket lock wait.
- F14: D3 has the relay take its lock and make its first creel read before it mails the notice, posts the line and exits.
  The D3 criterion names that order.
- F15: the `PATTERN-refusal-way-forward` constraint now covers the bot-scope list and the refused-read notice.
  The gate's `citation-path-missing` finding on that line is fixed: `docs/slack-app.md` is marked ` (new)`, since D10 creates it.
- F16: D5 serializes ensures per orch under a per-orch lock in the relay, with a D5 criterion.
  Fixed slightly differently: an ensure that finds the map now holds the channel treats it as held rather than stopping there.
  Stopping there would skip F7's invite of a held channel's operator.
- F17: D6 says where beam mail posts while the beams read fails, with a D6 criterion.
- F18: the D2 criterion now names the relay state, outbox and channel map.
- F19: the boundary rule's three sentences now sit in the `PATTERN-agent-addressing` constraint, and the Notes bullet is dropped.
  "invite the bot back" became "invite the bot into one", since F11 adds an invite into an adopted channel.
- F20: the third Notes bullet now keeps only the part F3 adds.
- F21: the support log's leanness 3 line says the archived-name risk moved to D5's rationale and the question ledger.

## Self-check

- fix-consistency, MEDIUM, D3's relay-start sentence contradicting the new missing-token order: fixed as proposed.
  `RequireConfigured`'s refusal now names only `lym slack start` and `lym slack away`, and the relay runs the check after its lock and first read.
  The fix-proposal lens raised the same finding at MEDIUM; merged here.
- fix-consistency, LOW, the D6 criterion keeping cursors unconditionally: fixed as proposed; it now keeps the cursors of an entry the map held.
- fix-consistency, LOW, support log's leanness 1 line pointing at the dropped Notes bullet: fixed as proposed.
- fix-claims, LOW, "records itself again at its next `lym creel inbox`" against the record interval: fixed as proposed.
  The wording now follows `hubregistry`'s own: "`lym creel inbox` in each hub, a minute on, records it again".
- fix-claims, LOW, "posts the line below" read as an unconditional post: fixed as proposed; the relay mails, then exits, and posts "first" only in the two stated cases.
- fix-way-forward, MEDIUM, moving the hubs record aside reopening the bypass: fixed with own change.
  The fix-proposal lens raised the same finding at MEDIUM; merged here.
  An absent hub cannot be told from a hub the record never named, so waiting for every hub is not buildable.
  Instead, D3's Bound states that the map test sees only listed hubs.
  D1 owes a resync hint to each newly listed hub and to every hub after a failed read, which also covers the dropped events.
- fix-way-forward, LOW, no catch-up once the live-hub list reads again: fixed by the same resync rule in D1 and the D1 criterion.
- fix-way-forward, LOW, no way back for relays a login ended while presence is terminal: fixed as proposed.
  Login names each hub whose relay it ended, with `lym slack start` as the way forward.
- fix-proposal, LOW, no stated route for login to end other hubs' relays: fixed as proposed.
  D3 names the route: each listed hub's relay record, as `lym slack stop` ends its own.
  `PATTERN-lym-files` now lets `lym slack login` reach those records through `hubregistry.Live`.
- fix-proposal, LOW, an operator invite Slack refuses for good retrying with no notice: fixed as proposed.
  A channel stands once its entry is written, so the prime's re-ensure does not loop.
  A refused invite mails one notice per episode naming `lym slack setup --operator <user-id>`, with a D5 criterion.
- fix-proposal, LOW, `last_slack_ok` changing meaning: fixed as proposed; `status` reports it as `last_healthy` (see F3).

## Disputed

None.

## Deferred

None.
