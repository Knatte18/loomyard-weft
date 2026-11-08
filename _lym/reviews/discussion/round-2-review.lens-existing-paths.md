```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I checked each state, file and invariant the record changes against the code paths that already write, read or reapply it.

**Paths traced, per decision:**
- D1, `RelayState.LastSlackOK` (stamped by `recordingSlack` on each Web API call) read by `starter.ensure`'s `lifetime` and by `status`'s `last_slack_ok`: **breaks** (F1).
- D1, a failed re-exec (`runInbound`'s `failed` callback, which retakes only the relay lock) after the holder released the machine socket lock: **breaks** (F3).
- D1/D2, `deps.CheckBinary` moving from after the inbound poll to after the outbound read: survives. The outbound read is bounded at 60 s.
- D1, `hubregistry.Live` as the list of hubs whose maps the holder reads: survives. A relay runs only with its hub's creel daemon up, and the daemon records the hub.
- D2, `presenceChanged`'s first read counting as no change, as the trigger for D5's "every listed beam when presence becomes slack": **breaks** in the `lym slack away` flow and on a restart while away (F2).
- D2, presence written by `state.WriteJSON` (atomic replace) under an `fswatch` names filter for `presence.json`: survives. The rename onto the name fires an event with that name.
- D2, hint files in the inbox meeting the "remove any other hint unread" rule, if the holder writes them with `fsx.AtomicWriteBytes` (temp file and rename): **breaks** (F4).
- D2, the channel map's own `internal/state` temp and lock files: survive. They sit in `.lym/slack/` and `locks/`, outside the inbox and the presence filter.
- D3, `slack.json` rewritten by an older binary's `WriteToken` (`tokenRecord{Token}`): survives, because `state` keeps the unknown keys. An old binary reads only `token`.
- D3, `RequireConfigured` as the gate of `EnsureRelay` and `EnsureRelayWhileAway`:
  - `orchcli` `relayStarter`, the `fabriccli` anchor `StartRelay` and the `beamcli` `ensureSlackRelay` tick all survive with a quiet `OutcomeNotConfigured`.
  - `slackcli` `start`/`away` survive.
- D3, a relay missing a token at its start: `wiring.go` `runRelay` refuses at line 188, and `RunRelay` refuses at line 186 before any lock, mailbox or first read. Both run before the notice D3 wants, against the "first creel read before any Slack call" invariant: **breaks**, ordering not stated (F5).
- D3, `slackcli` `setup` reading only the bot token: survives.
- D5/D8, dobby wait reminders posting as `FormatScopedProgram(shortname, slug, DOBBY)`, with the beam slug, empty for the prime: survives. They route to the beam's or the prime's channel.
- D6, `stopRelay`: presence terminal, then `StopRelay`, then the new archive. Its retry message and dobby's terminal-presence gate survive. An orch bring-up's `EnsureRelay` in the gap reopens only on mail, by design.
- D8, `outbound.refused`/`reportStale`, now per message channel, with the outbox keyed by message id: survive.
- D10, `setup.json` with the old `channel_id`, read by `status` (changed to list the map) and the old `inbound`/`outbound` (replaced): survives. The leftover `cursor.json` is unread.
- D10, existing ways forward that name the old setup model or old scopes:
  - `scopeRefusal` with `botScopes`, `cli.go:31` and `cli.go:330`;
  - `inbound.readFailed`'s "`lym slack setup` for a channel the bot cannot read", `inbound.go:297`.
  
  These are outside the rewording constraint: **breaks** (F6).

**Findings:**

F1 — MEDIUM, design, D1/D2 against `starter.ensure` (Notes: "dobby's restart … keep their logic").
- **Problem:** with the poll gone, an idle relay makes no Web API call, so `LastSlackOK` stops moving soon after the start read. A non-holder relay makes none at all.
- **Effect:**
  - `lifetime` reads a relay that ran idle for hours and then died as having died young.
  - Five of those in a row make `EnsureRelayWhileAway` answer `OutcomeGaveUp`, and dobby stops restarting the relay.
  - `status`'s `last_slack_ok` stops meaning "the relay is healthy".
- **Gap:** the record keeps that logic unchanged and names nothing that stamps liveness, such as the socket's pong or a periodic mark.
- **Evidence:** `internal/slackengine/start.go:238-245` (`lifetime`) and `:213-220`; `relay.go:117-133` (`recordingSlack.stamp` wraps the Web API only); `slackcli/cli.go:529-535`.

F2 — MEDIUM, design, D5 "every listed beam's channel when presence becomes `slack`, seen at the next wake".
- **Problem:** the existing presence-change detection treats its first read as no change. `lym slack away` spawns the relay, waits only for its first creel read, then writes presence. That write lands before the relay's first presence read whenever the start's Slack reads run first.
- **Effect:**
  - The change is never seen, and no beam channel is ensured, even ones `lym slack stop` archived.
  - A relay that dobby restarts while presence is already `slack` never ensures beam channels either.
- **Gap:** the record states no presence baseline, such as the presence before the start or a "presence is slack at start" rule.
- **Evidence:** `inbound.go:253-263` (`presenceChanged`); `slackcli/cli.go:426-446` (`away` order); `relay.go:216-225`.

F3 — LOW, design, D1 re-exec.
- **Problem:** "Before a re-exec … releases the machine socket lock; after the exec it waits its turn." The existing failed-exec callback retakes only the relay lock.
- **Effect:** a relay whose exec fails keeps running without the socket lock and without rejoining the lock wait. When it was the machine's only relay, push stops until a restart.
- **Gap:** the record does not say the failed path re-queues for the socket lock.
- **Evidence:** `relay.go:330-339`.

F4 — LOW, design, D1/D2 hint files.
- **Problem:** the drain removes "any other hint unread". A hint written through the repo's atomic write (`state` → `fsx.AtomicWriteBytes`, temp file and rename in the same directory) leaves a temp entry in the inbox. The drain can remove that entry mid-write.
- **Effect:** the rename fails. The hub is then owed a resync, and its orch gets a spurious `[slack relay]` notice.
- **Gap:** the record does not say how a hint is written, for example a plain create, or a temp file outside the inbox. It also does not say which names the drain ignores.
- **Evidence:** `internal/state/state.go:73-100`; decision-record D2 lines 107-109.

F5 — LOW, consistency, D3 missing-token notice.
- **Problem:** the existing relay start refuses on a missing token before it builds its mailbox, takes its lock or makes its first creel read. D3 has the relay mail the orch and post to the prime's channel before it exits.
- **Effect:** the record does not place that notice after the relay lock and the first read. As written, it is a Slack call before the first creel read, which the relay's documented invariant forbids.
- **Evidence:** `slackcli/wiring.go:188`; `relay.go:179-187`; `doc.go:55`.

F6 — LOW, design, D10 and the `PATTERN-refusal-way-forward` constraint.
- **Problem:** the rewording covers only ways forward that name `lym slack setup --channel`. Two existing texts point at the removed model:
  - `scopeRefusal`/`botScopes` name the old scope set, including the private-channel `groups:*` scopes, and not `channels:manage` or `chat:write.customize`.
  - `readFailed` sends a refused read to "`lym slack setup` for a channel the bot cannot read".
- **Effect:** under D10 both misdirect the operator.
- **Evidence:** `slackcli/cli.go:31`, `:195-197`, `:330-332`; `inbound.go:297`.

**Tool calls:**
1. Grep for other senders to `SLACK` and for `state`'s temp naming: D8's routing of non-orch senders, not in the manifest.
2. `beamcli/dobbydeps.go` 255-300 and the `state` write grep: the dobby reminder's sender, and how `state` writes.
3. `creelcli` `NewScopedProgramClient`: the reminder's program name form.
4. `beamcli/dobbydeps.go` 225-252: the source of the reminder's slug.
5. `internal/state/state.go` 73-110: the atomic write path the hint files would meet.
