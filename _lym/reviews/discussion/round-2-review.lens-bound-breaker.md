```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens, attacking every bound, guard and claim about existing code in the slack-machine-bridge decision record.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: no agent name holds `::` today, so existing parsers reject it | held |
| D1: the own-name check runs before `CheckName`, so a former name that fails the name rule is still refused (`machinename.go:76-83` keeps such former names) | held |
| D2: one new edge only, orch to orch; every other sender refused at post | held |
| D3: the scheduler's start scan and wake skip machine-outbound; the take verb is its only reader | held (a decision, no existing reader breaks it) |
| D3: a lost settle is made again at the relay's start from the bridge-post record | held |
| D3: a late settle-delivered for a returned message gets a `CREEL` notice; a repeated settle-delivered for a delivered one is a no-op (today `settleDelivered` ignores non-pending, `creelengine/outbound.go:127`) | held |
| D3 bound: program-claim forgery from an orch pane or a no-pane process (off Linux, any process) | held as stated, not re-verified |
| D4: both sides compute the same name, cut at 80 with a hash as `ChannelName` cuts (`channelname.go` `fitName`) | held |
| D4 bound: collisions need two machine names overlapping at a hyphen | held; shortnames forbid `-` (`agentname.go:88-95`), so only machine names or hyphenated beam slugs can overlap, and the creator and envelope checks still guard |
| D4: `name_taken` uses `FindChannel`, a paged `conversations.list` (`api.go:278`) | held |
| D4: an archived or gone channel is dropped from the map, and the next send creates or finds it again | **broken**, F2 |
| D4/D8: an adopted channel reads from its start, and dedupe keeps a reread from delivering twice | **broken** past creel's retention, F3 |
| D5: the envelope rests on `chat:write` alone | held |
| D6: bot users are named `lym-<machine>` | held (the manifest sets `display_name: lym-<machine>`, `docs/slack-app.md:16`) |
| D6: in a bridge channel, other operator text is answered once | **broken**, F1 |
| D6: `revoke` removes trust | **broken** while a held message sits before a stale `approve`, F1 |
| D7: a restart never reposts once the bridge-post record is written, and the record is apart from the outbox | held |
| D7: five refusals return the message, "as the relay drops orch mail today" | **broken** if the counts share the outbox record, F4 (`refusalCap = 5`, `outbound.go:32`) |
| D8: the holder routes bridge-map channels; a foreign event becomes a rate-capped bridge-resync hint | held |
| D8: a resync takes the local orch only from this bot's or a pinned app's envelope | held |
| D8: a carried record with an unknown `of` folds to an id only | held as far as `applyCarried` shows (`fold.go:569-600`) |
| D10: an event only speeds the wake; the poll carries the mail | held |
| D11: the manifest holds `channels:manage`, `channels:join`, `channels:read`, `users:read`, `chat:write`; the relay already calls create, invite, join and archive (`api.go:263-333`); `bots.info` exists as `BotAppID` (`api.go:248`) | held |

**Findings**

**F1 — BLOCKING, design: D6/D8, operator commands past a held message**
- **Location:** `decision-record.md:115-116`, `:168-169`, `:181`.
- **Summary:** operator commands written after a held peer message are re-read on every poll. A stale `approve` then undoes a later `revoke`, and "answered once" fails.
- **Counterexample:**
  1. A peer message is held for `button-off`. The delivery cursor stays before it (D8:181).
  2. The operator writes `approve osl-1046` in the bridge channel. It is handled "past a held peer message" (D8:169), but the cursor cannot advance past the hold.
  3. Later the operator writes `revoke osl-1046` in the orch channel.
  4. The next bridge poll re-reads the bridge channel from the held cursor and handles the old `approve` again. The pin comes back, and the peer is silently re-trusted.
- Under the same hold, any other operator text is "answered once" (D6:115) yet re-read and answered on every poll.
- The record never says how handled messages after a held cursor are remembered.
- **Evidence:** the existing model leaves the cursor before an unhandled message (`internal/slackengine/inbound.go:698-715`).

**F2 — MEDIUM, design: D4, a hand-archived bridge channel**
- **Location:** `decision-record.md:91`, `:81-84`.
- **Summary:** when the operator archives a bridge channel in Slack, the next send neither creates nor finds a usable channel.
- **Counterexample:**
  1. An archived channel keeps its name, unlike retirement, which renames under `ArchivedName` first.
  2. The drop leads to create, which returns `name_taken`.
  3. `FindChannel` returns the archived channel. D4's adopt names only the creator check, so it either adopts it and posting fails and drops it again in a loop, or does something D4 leaves unstated.
- **Evidence:** the existing `adopt` refuses an archived channel with a "rename that channel" way forward (`internal/slackengine/keeper.go:194-197`), and D4 states no such outcome for a bridge.

**F3 — MEDIUM, design: D4/D8, rereading from the start against a seven-day dedupe window**
- **Location:** `decision-record.md:85-86`, `:151-155`, `:183`.
- **Summary:** an adopted or resync-added channel reads from its start, but dedupe holds only for ids creel still has. Creel retires journal days after `journalRetention = 7 * 24 * time.Hour`.
- **Counterexample:**
  1. The hub's never-tracked `.lym/slack/` bridge map is lost, for example by a re-clone or a wipe.
  2. The startup resync re-adds every bridge channel the bot is in.
  3. Months of history are delivered again as new mail to the orch.
- **Evidence:** `internal/creelengine/journal.go:68`; the dedupe and id-clash window is the seven-day retention (`internal/creelengine/receive.go:238-253`).

**F4 — LOW, consistency: D7, where the refusal count lives**
- **Location:** `decision-record.md:136`, `:139-142`.
- **Summary:** D7 does not say where the five-refusal count for bridge mail is kept.
- **Counterexample:** today's counts live in the outbox record (`internal/slackengine/outbound.go:363`). `read` replaces that record with an empty one after every confirmed orch-mail read (`outbound.go:107`). Kept there, bridge counts reset with every orch-mail read, and the five-refusal return never fires before the hour.

**Tool calls**
1. `outbound.go` `refused`/`dropEntry` and `channelname.go`: the refusal-cap mechanics and the sanitizer and cut rules were not in context.
2. `api.go` method list and `docs/slack-app.md` scopes and bot name: needed to check D11's scope claims and D6's bot-name claim.
3. `creelengine` carried-record grep, `machinename` and `agentname` name rules: needed to check D8's carried-fold claim and D4's collision bound.
4. `fold.go` `applyCarried`, `machinename.go` `CheckName`/`Names`, `agentname.go` `ValidateShortname`: the bodies behind call 3's hits.
5. `receive.go` dedupe, journal retention and `refusalCap`: needed to check D8's "never delivers twice" claim and D7's cap.
