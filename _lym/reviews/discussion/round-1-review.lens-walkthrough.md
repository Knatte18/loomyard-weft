```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism end to end on one concrete case and checked the end state against the Decision's stated purpose.

**Decisions walked**

- **D1**, an event for hub B's channel read by hub A's holder: held. A finds B's map through `hubregistry.Live`, writes the hint, acknowledges, and B reads Slack. The wake on B's side breaks; see finding 1.
- **D2**, an operator message in the prime channel woken by its hint: broken. The hint lands in an inbox directory under `.lym/slack/`, and the watch is on the Slack directory (finding 1). The relay's own writes wake it (finding 2), and the thread-cursor rules replay old replies (findings 3 and 4).
- **D3**, two cases: a machine upgraded with only a bot token, and a login with an `xapp-` token. The `xapp-` login holds. The upgrade case is unclear: it is never said whether `EnsureRelay`'s outcome becomes an error or stays `OutcomeNotConfigured` (finding 5).
- **D4**, three cases: `ly` / `neve` / slug `prime` gives `ly-neve-prime_beam`, `prime-beam` stays distinct, and an over-long name gets the hash suffix. Purpose met.
- **D5**, two cases: the prime channel at a first start (create, map entry with an empty cursor, invite), and `name_taken` on an archived channel (list, then D6 reopen, then invite). Both met. Adopting a live channel replays old thread replies (finding 3).
- **D6**, two cases: a beam that landed is archived at the next outbound-wait wake, and a stop, start and `away` run reopens through the user token. Purpose met.
- **D7**, two cases. A message in a beam channel is routed through the map: met. A `thread_broadcast` is forwarded twice (finding 4).
- **D8**, three cases. Mail from `ly:foo:orch` reaches the `foo` channel under its name and icon: met. Retries after `is_archived` and `channel_not_found` are bounded by the existing `refusalCap` (`internal/slackengine/outbound.go:32`): met. A bot removed from its channel is not covered (finding 6).
- **D9**, `away` in a beam channel: presence is set, every orch is told, and the answer goes to that channel. Purpose met.
- **D10**, `setup --operator`: `users.info` runs, and an old `setup.json` keeps its channel key. The scopes cover every call the Decisions make. Purpose met.
- **D11**, the seam and the `httptest` run: purpose met.

**Findings**

1. **BLOCKING · design · D1 and D2, hint inbox and wake source.** A hint is written into "that hub's inbox directory under `.lym/slack/`" (decision-record.md:40). The relay wakes on "an `internal/fswatch` event on its Slack directory" (decision-record.md:83). `fswatch.Watch` adds only `dir` itself, so it is not recursive (`internal/fswatch/fswatch.go:41`). A hint file in a subdirectory therefore raises no event. The message waits for the outbound read's wait to end, up to 60 s plus backoff (`internal/slackengine/outbound.go:25`), which misses the Goal's "within about a second" (decision-record.md:7). The record has to put the hints in the watched directory, or watch the inbox directory itself.

2. **BLOCKING · design · D2, wake source with no floor and no filter.** Every fswatch event on the Slack directory wakes the relay, and every wake drains the inbox and runs D6's archive and D5's presence check (decision-record.md:83, 146, 171). The relay itself writes to that directory:
   - `relay.json`, after every successful Slack call (`internal/slackengine/relay.go:125-131`);
   - `relay.log`, on every failure (`internal/slackengine/relay.go:92`);
   - `outbox.json`, the cursors in `channels.json`, and lock files.

   Concrete case: the archive of a beam that is no longer listed is refused, for example over a missing scope. The failure appends to `relay.log`, that fires an event, the next wake retries the archive, and it fails again. Each round is a Slack call, with no one-second floor and no backoff. That breaks PATTERN-daemon-wakeups ("at most once a second") and the operator's low idle cost. The record names neither a filter on watched names (`fswatch.Watch` takes `names...`) nor a wake floor.

3. **MEDIUM · design · D2, a missing thread cursor reads the whole thread.**
   - Case (a), pruning: a thread cursor is pruned after 24 hours with no reply (decision-record.md:72). A thread hint then reads replies "after its thread cursor" (decision-record.md:79), so a pruned or never-set cursor reads from the thread's start. The operator replies on day 1, then again on day 3 in the same thread: the day-1 reply is forwarded a second time.
   - Case (b), adoption: a live channel adopted on `name_taken` gets only its top-level cursor set to its newest message (decision-record.md:154). The start scan then takes every message of the last 24 hours that "has no cursor" (decision-record.md:80), and forwards every operator thread reply of that day again.

   Neither case meets D2's purpose, "duplicates harmless" (decision-record.md:86).

4. **MEDIUM · design · D2 and D7, `thread_broadcast` forwarded twice.** D7 admits `thread_broadcast` (decision-record.md:198). A broadcast reply is in the top-level history and in that thread's replies at the same time. A start or resync read forwards it through `conversations.history` after the top-level cursor. The same read's `latest_reply` scan then reads the thread's replies, which hold that message, and forwards it again. The two cursors are kept apart, so nothing dedupes it. That contradicts "forwarded once" (decision-record.md:87) and D7's acceptance bullet (decision-record.md:346).

5. **MEDIUM · consistency · D3, "refuses" against today's not-configured outcome.** D3 says "A machine missing any refuses before a relay is spawned" (decision-record.md:102), and the Notes say only the shared check changes (decision-record.md:371). Today, `ensure` answers `OutcomeNotConfigured` quietly for a machine with no token (`internal/slackengine/start.go:180-181`). Two callers turn an error from it into something visible:
   - the orch bring-up turns it into a warning (`internal/orchcli/start.go:550`);
   - `StartRelay` in the anchor change returns it as an error (`internal/fabriccli/anchordeps.go:282-283`).

   If "refuses" means `EnsureRelay` returns an error, every orch start and anchor change on a machine without Slack warns or fails. If it means `OutcomeNotConfigured`, the refusal that names the missing kind never reaches the operator on those paths. The record has to say which.

6. **LOW · design · D8, a bot removed from its channel.** D8 recovers from `is_archived` and `channel_not_found` only (decision-record.md:219-220). If the operator removes the bot from a channel, every post is refused with another code, probably `not_in_channel`. The record names no recovery: the map still holds the channel, so D5 never invites the bot back. Each message is then dropped after `refusalCap` refusals. The "dropped" line posts to that same channel (decision-record.md:218), so it fails too, and only a Warn log line is left (`internal/slackengine/outbound.go:197-199`).

7. **LOW · design · D2, the re-exec check loses its loop.** Today, `CheckBinary` runs after each inbound poll (`internal/slackengine/relay.go:330-339`). D2 removes the poll loop and names no point where a relay checks for a replaced binary or re-execs. That includes how the machine socket lock is given up and taken back around the exec, as `relayLock` does today (`internal/slackengine/relay.go:60-80`).

**Tool calls**

- Read `internal/orchcli/start.go:550-590` and `internal/fabriccli/anchordeps.go:270-300`, to see how `EnsureRelay`'s callers handle its error. My context held only those callers' grep lines.
