```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guards, guarantees and claims about existing code.

**Claims attacked**

1. D1 rationale, "one connection per machine is the only way one hub never gets another's events": **broken** (finding A).
2. D1 bound, "a planted file can at most make a relay read its own channel": **broken** (finding B).
3. D1, a failed hint write is retried by Slack, and the holder's stall is bounded: **broken** (finding C).
4. D1 and D2, the hint's fswatch wake: **broken** (finding D).
5. D2, "a message both hinted and caught up is forwarded once", with thread cursors pruned: **broken** (finding E).
6. D2 against D7, "forwarded once" for `thread_broadcast`: **broken** (finding F).
7. D2, the gaps push leaves are only a stopped relay, the lock hand-over and a reconnect: **broken** (folded into C).
8. D2, "cursors only move forward" makes concurrent reads safe: held for the cursor itself. Two concurrent drains could still forward twice (folded into E as a note).
9. D3, the user token is used only on this hub's own channels, held by no other orch's entry in any live hub's map: **broken** (finding G).
10. D3, the guard's result when another hub's map is unreadable: **broken**, it is unstated (finding H).
11. D3, "the relay start and `RequireConfigured` apply one check", and the Notes' "the configured check they share": **broken** (finding I).
12. D3, "`EnsureRelay` and dobby's restart never spawn a relay that refuses at once", across the upgrade re-exec path: **broken** (finding J).
13. D3, "keeping the `token` key lets an older binary still read the bot token", and an old login keeping the new keys: held. `internal/state/keep.go:1-2` keeps undeclared keys.
14. D4, "`prime_beam` cannot come from the slug grammar": held (`internal/agentname/agentname.go:98-110`, no `_`).
15. D4, "the machine segment keeps each machine's app on its own channels": **broken** (folded into G).
16. D5 bound, the adopt race needs duplicate shortnames: held as stated.
17. D5, the notice names `lym fabric shortname` as the way forward: **broken** (finding K).
18. D6, "archiving after the relay has ended means no live relay can reopen a channel the stop has just archived": held in effect. `EnsureRelay` from a prime bring-up can start a relay between the stop and the archive, but D8's `is_archived` reopen repairs that, so no defect.
19. D6, "a failed `Hub` read archives nothing": held.
20. D7, the operator-only and bot-skip guards hold in every channel: held. The operator id is per hub, and the bot skip is kept.
21. D8, `is_archived` and `channel_not_found` re-posts are bounded by "the refusal cap": held. The cap is the existing `refusalCap` of 5 (`internal/slackengine/outbound.go:31-32`), counted per message.
22. D10, "a `setup.json` written by an older binary keeps its channel key, which nothing reads": held (`internal/state/keep.go`).
23. Constraints, "the lock wait blocks without waking": held. `lock.AcquireWriteLock` blocks on flock (`internal/lock/lock.go:30-40`).

**Findings**

**A. MEDIUM · design · D1, D3 (login)**
Nothing checks that the three tokens belong to one app and one workspace.
- `lym slack login` confirms each token alone: `auth.test` for the bot and user tokens, `apps.connections.open` for the app-level token.
- Counterexample: the operator pastes osl's `xapp-` on neve. Neve's holder joins osl's app's socket, and Slack spreads osl's events across both connections.
- Neve's holder finds those channels in no live map on neve. It acknowledges them and drops them (D1: "acknowledged and dropped with a Debug log line"), so osl loses about half its events until its next reconnect.
- A user token from another workspace breaks the reopen path in the same way.
- Fix: login checks that the app-level token's `app_id` (from the `hello` envelope's `connection_info`) and team match the bot token's.
- Evidence: decision-record.md:98 and :44; `internal/slackcli/cli.go:223` (`AuthTest` only).

**B. MEDIUM · design · D1 bound, D2**
The record never says a relay checks a hint's channel against its own map before it reads.
- D1's bound ("a planted file can at most make a relay read its own channel") depends on that check, and on rejecting a malformed name. Neither is stated.
- D2 reads `conversations.history` "for a hint for that channel" after "its cursor". A channel outside the map has no cursor, which per D5 means the channel's beginning.
- The bot is a member of every hub's channels on the machine. A planted `C&lt;other hub's channel&gt;` hint could therefore make relay A read hub B's channel from the start. D7 does not say where an unmapped channel's text goes.
- Fix: a hint whose channel or thread is not in the relay's own map, or whose name does not parse, is removed unread.
- Evidence: decision-record.md:41-43, :58, :73-77, :150.

**C. MEDIUM · design · D1 (holder stall, ack), D2 (gap list)**
A holder that stalls without ending keeps the machine lock, and no bound is stated.
- Per event, the holder reads every live hub's map under that map's `internal/state` lock. A map lock held long by another process stalls every hub's push. The ping is in the same holder, and nothing says a stalled hint write ends the connection.
- "A failed hint write leaves the envelope unacknowledged, so Slack retries it" is bounded only by Slack's retries. After they run out, the hint is lost. D2's catch-up then waits for the next message or for a reconnect, which the Constraints put at "every few hours".
- D2's list of gaps ("a relay not running, the lock hand-over and a reconnect") leaves out this lost hint.
- Fix: a time limit on a hint write. A holder over it, or with retries exhausted, reconnects, and the resync on that new connection covers what was lost. Add the case to D2's gap list.
- Evidence: decision-record.md:39-45, :88-89, :289; `internal/lock/lock.go:30-40`.

**D. MEDIUM · design · D1 and D2 (wake source)**
The hint may land in a directory the watch does not see.
- D1 writes hints into "that hub's inbox directory under `.lym/slack/`". D2 wakes on "an `internal/fswatch` event on its Slack directory".
- `fswatch.Watch` adds one directory and does not recurse (`internal/fswatch/fswatch.go:36-44`). A hint in a subdirectory `.lym/slack/&lt;inbox&gt;/` fires no event on `.lym/slack/`.
- The relay then wakes only at the end of the outbound read: a `ReadWait` of up to 60 s plus a wait of up to 60 s (`internal/slackengine/outbound.go:25`, `internal/slackengine/backoff.go:15`). That misses the Goal's "within about a second".
- The same slow fallback applies when `fswatch.Watch` fails (for example, inotify instances run out), and the record states no bound for it.
- Fix: state that the watch covers the inbox directory itself, and state the bound when the watch fails.
- Evidence: decision-record.md:40, :83, :6-7.

**E. MEDIUM · design · D2 (thread cursor prune)**
Pruning a thread cursor re-forwards the thread.
- Counterexample: the operator replies in thread T on day 1, and the reply is forwarded and the cursor set. T gets no reply for 24 h, so its cursor is pruned.
- On day 3 the operator replies in T. The thread hint reads `conversations.replies` "after its thread cursor", but no cursor exists, so every earlier reply in T reaches the orch again. That breaks "forwarded once" and can replay old operator instructions.
- Note: the record also never says inbox drains (fswatch wake against outbound-wait wake) are serialised. Forward-only cursors do not stop two concurrent drains from forwarding one message twice.
- Fix: a read with no thread cursor starts after the thread's last reply as of the prune, or after the top-level cursor. Also state that drains are serialised.
- Evidence: decision-record.md:72, :79, :82-83, :87.

**F. MEDIUM · design · D2 against D7 (`thread_broadcast`)**
A `thread_broadcast` is forwarded twice.
- It appears as a top-level message in `conversations.history` and as a reply in `conversations.replies`. D7 admits it, and D2 reads both routes with separate cursors, so the orch gets it twice.
- Today's code skips every message whose `ThreadTS` differs from its `TS` (`internal/slackengine/inbound.go:309`). The record admits the subtype without saying which route forwards it.
- Fix: forward a broadcast on one route only, and skip it on the other.
- Evidence: decision-record.md:79, :198, :346.

**G. MEDIUM · design · D3 bound, D4, D5 step 2, D6**
Channel names can collide across machines, and the user token then reopens another machine's channel.
- The guard ("held by no other orch's entry in any live hub's map") sees only this machine's hubs, because `hubregistry.Live` reads a per-machine record (`internal/hubregistry/hubregistry.go:97-116`).
- D4 does not make names unique across the workspace:
  - `machinename.FromHostname` keeps only the part before the first dot, so `neve.home` and `neve.office` both read `neve`.
  - `-` separates segments and is also legal inside them. Machine `neve` with slug `a-b` and machine `neve-a` with slug `b` both give `ly-neve-a-b`.
- On `name_taken`, D5 reopens an archived channel of that name with the user token and invites the bot. Machine 2 can then unarchive and join machine 1's archived beam channel, so two machines relay one channel. The board's "two hubs or two machines never relay one channel" breaks.
- Fix: one option is to reopen only a channel whose creator, or whose members, include this machine's bot.
- Evidence: decision-record.md:106, :123-124, :131, :151-153; `internal/machinename/machinename.go:19-22`.

**H. MEDIUM · design · D1, D3, D5**
The record never says what an unreadable or undecodable map of another live hub does.
- D1's routing and D3/D5's "no other orch's entry in any live hub's map" test both depend on reading every live hub's map.
- Treated as empty, an unreadable map lets the user token unarchive or adopt that hub's channel. Treated as an error, D1 does not say whether the event is left unacknowledged, which would retry against every hub's flow, or dropped.
- Fix: an unreadable map fails the guard closed (refuse), and D1 states its acknowledgement rule for that case.
- Evidence: decision-record.md:39, :106, :152.

**I. LOW · consistency · D3, Notes bullet 3**
"The relay start and `RequireConfigured` apply one check" and "only the configured check they share changes" both misstate the code.
- `ensure` does not call `RequireConfigured`. It reads the token and the setup itself and answers `OutcomeNotConfigured` with no error (`internal/slackengine/start.go:172-182`).
- Its callers rely on that silence. `fabriccli`'s `StartRelay` discards the outcome (`internal/fabriccli/anchordeps.go:282-283`), and orch bring-up starts the relay on every prime (`internal/orchcli/start.go:557-567`).
- So "refuses before a relay is spawned, naming `lym slack login` and the missing kind" either makes every prime bring-up or anchor change refuse on a machine that lacks one token, or it is not a refusal at all. The record has to say which path refuses and which stays a silent `not-configured`.
- Evidence: decision-record.md:101-103, :371.

**J. MEDIUM · design · D3 (upgrade)**
An upgrade kills a running relay while the operator is away, and nobody is told.
- A relay running when the binary is replaced re-execs onto it (`internal/slackengine/relay.go:330-339`). The new image's `RunRelay` runs `RequireConfigured` first (`internal/slackengine/relay.go:186`).
- On a machine with only the old `xoxb-`, the relay exits at once. Dobby's `EnsureRelayWhileAway` then answers `OutcomeNotConfigured` silently every tick (`internal/slackengine/start.go:180-181`, `internal/beamcli/dobbydeps.go:361`).
- Presence stays `slack`, the operator reaches nothing, and no orch is told. "Never spawn a relay that refuses at once" does not cover the re-exec path.
- Fix: name who is told. For example, the exiting relay mails the hub's orch a `[slack relay]` notice naming `lym slack login` and the missing kind.
- Evidence: decision-record.md:101-103.

**K. MEDIUM · consistency · D5 step 2**
The way forward D5 names cannot work.
- When another hub's map holds the channel, D5's notice names `lym fabric shortname` as the way forward. That verb refuses to change a recorded shortname: "refusing to change it, since that would orphan every name already in use" (`internal/fabriccli/shortname.go:39-43`).
- `hubregistry`'s own refusal says the fix for duplicate shortnames is "remove the extra clone, since a recorded shortname never changes" (`internal/hubregistry/hubregistry.go:93`).
- Fix: name a way forward that works, such as removing the duplicate clone.
- Evidence: decision-record.md:156.

**Tool calls**

1. A grep and sed over `internal/fabriccli` for the shortname verb, `internal/orchcli/start.go` and `internal/fabriccli/anchordeps.go` around `EnsureRelay`, and `internal/state/state.go` and `internal/state/keep.go`. The manifest lacked the `EnsureRelay` callers' outcome handling and whether `internal/state` keeps unknown keys.
2. `internal/fabriccli/shortname.go` lines 1-60, to check whether `lym fabric shortname` can change a recorded shortname, the way forward D5 names. The manifest did not include it.
