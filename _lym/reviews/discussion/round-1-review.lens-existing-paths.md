```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: tracing the code paths that already write or read the state each decision changes.

**Paths traced, per decision:**

- **D1/D2, hint wake.** The `internal/fswatch.Watch` watch on the Slack directory against hints written to "that hub's inbox directory under `.lym/slack/`": **breaks** (F1).
- **D1/D2, own writes in the watched directory.** The relay writes `relay.json` (`recordingSlack.stamp`, on every successful Slack call), `outbox.json`, `channels.json` and `relay.log`, all into the watched Slack directory: **breaks** (F3).
- **D2, re-exec onto a replaced binary.** `deps.CheckBinary` runs after each `poll` in `runInbound`, and the poll goes: **breaks** (F2).
- **D3, a live relay re-execs after the upgrade.** On a machine holding only today's bot token, the new image's `RunRelay` reaches `RequireConfigured`: **breaks** (F4).
- **D3, starts at the upgrade.** `EnsureRelay` from prime bring-up (`orchcli/start.go:566`, `fabriccli/anchordeps.go:282`) and dobby's `EnsureRelayWhileAway` (`beamcli/dobbydeps.go:361`), through `ensure`'s own token and setup reads (`start.go:172-182`): they start nothing and answer `OutcomeNotConfigured` with no refusal. This **survives** (no bad spawn) but feeds F4.
- **D3, a token file rewritten by an older binary's `login`.** `WriteToken` goes through `state.WriteJSON`, which keeps unknown keys (`doc.go:13`): **survives**.
- **D5/D8, dobby's wait reminders.** They post as `NewScopedProgramClient(worktree, slug, dobbyProgram)` (`dobbydeps.go:281`), so the sender is `&lt;sn&gt;:&lt;slug&gt;:DOBBY` and routes to the beam's channel, which it creates on first mail: **survives**.
- **D7/D8, the orch role contract.** `contracts/orchrole/role-slack.md:4-6,22-23` still describes one channel and the `@&lt;slug&gt;` answer hint: **misreads** (F6).
- **D8, the stale-mail line.** `outboxRecord.Stale` is one hub-wide count, posted once by `reportStale` (`outbound.go:136-147`); D8 wants each line in "that message's channel": **breaks** (F5).
- **D8, the refusal cap.** `refused` counts every `*APIError` in `outbox.json` `Refusals` against `refusalCap` (`outbound.go:184-201`); `is_archived` and `channel_not_found` are API errors: **survives**.
- **D9, presence readers.** The orch prompt (`orchengine/prompt.go:40`), the dobby `Away` check (`dobbydeps.go:265`) and `SetPresence`: **survive**, since presence stays one file per hub.
- **D10, setup readers.** The `ReadSetup` callers are `inbound.poll`, `outbound.read`, `RequireConfigured`, `ensure` and `status`. None reads `setup.json` outside slackengine and slackcli. **Survives** as far as the record covers them. Their way-forward strings still name `--channel` (`inbound.go:228`, `outbound.go:81`, `credentials.go:87`); the record's refusal constraint covers only new refusals (F7).
- **D6, `lym slack stop`.** `stopRelay` sets presence to terminal, ends the relay through `runproc`, then archives from the CLI. After that, dobby's `EnsureRelayWhileAway` answers `OutcomeTerminal` and starts nothing: **survives**.
- **D4/D5, an operator-made `ly-neve-prime` channel already in use.** The create hits `name_taken`, and the channel is adopted if the bot is a member: **survives**.

**Findings:**

- **F1 — BLOCKING, design. D1/D2: the watch is not on the directory the hints go to.**
  D1 writes each hint into "that hub's inbox directory under `.lym/slack/`", a subdirectory. D2 wakes on "an `internal/fswatch` event on its Slack directory".
  `fswatch.Watch` adds one directory to fsnotify, which does not recurse, so a file created in the inbox subdirectory raises no event on `.lym/slack/`. Every hint then waits for the outbound read's wait to end, up to 60 s (`outboundReadTimeout`), which defeats the Goal's one-second delivery.
  The Decision should name the watched directory as the one the hints land in.
  Evidence: `internal/fswatch/fswatch.go:36-43`, `internal/slackengine/outbound.go:25`, `decision-record.md:40,83`.
- **F2 — MEDIUM, design. D2: removing the poll strands the binary-replace re-exec.**
  `runInbound` calls `deps.CheckBinary` after every poll. That call is the relay's only route onto a replaced lym binary, and it also releases and retakes the relay lock. D2 removes the poll, and the record names no loop that now runs the check. The Notes list only the outbound read, the ping and reconnect, and the inbox watch.
  The record also says nothing about the machine socket lock and the open connection across that exec.
  Evidence: `internal/slackengine/relay.go:182-184,330-339`, `decision-record.md:370`.
- **F3 — MEDIUM, design. D2: the watch on the Slack directory wakes on the relay's own writes.**
  `recordingSlack.stamp` rewrites `relay.json` after every successful Slack call. The outbox, channel map and relay log are written into the same directory. With no name filter, each of those writes is a wake, and each wake drains the inbox and runs D6's `slackengine.Hub` read.
  So every hint costs at least one further wake caused by its own read. The record states neither a name filter (`fswatch.Watch` takes one) nor that this bound is intended. This bears on the idle-cost wish and on `PATTERN-daemon-wakeups`.
  Evidence: `internal/slackengine/relay.go:125-133`, `internal/slackengine/geometry.go:13-28`, `internal/fswatch/fswatch.go:33-35`, `decision-record.md:83,171`.
- **F4 — MEDIUM, design. D3: on upgrade, a live relay re-execs onto the new binary and exits silently.**
  On a machine holding only today's bot token, the new image's `RunRelay` fails `RequireConfigured` on the missing `xapp-` or `xoxp-` token and returns. The refusal lands only in `relay.log`. Dobby's restart then answers `OutcomeNotConfigured`, which logs nothing.
  Presence stays `slack`, so the orchs keep posting to `&lt;sn&gt;:SLACK` and the operator hears nothing. D3's "naming `lym slack login` and the missing kind" reaches no one on this path.
  The record should state who is told and how, such as a `[slack relay]` notice to the hub's orch, or presence set back to terminal.
  Evidence: `internal/slackengine/relay.go:186-188`, `internal/slackengine/start.go:172-182`, `internal/beamcli/dobbydeps.go:361-372`, `decision-record.md:101-103`.
- **F5 — LOW, design. D8: the stale-mail line cannot go to "that message's channel".**
  The outbox record keeps stale mail as one count, and `reportStale` posts one aggregate line. Posting per channel needs a per-sender or per-channel count in `outbox.json`. The record states this neither as a change nor as the line's channel.
  Evidence: `internal/slackengine/outbound.go:43-47,136-147`, `decision-record.md:218`.
- **F6 — LOW, consistency. D7/D8: the orch role contract still tells orchs about the one channel and the `@&lt;slug&gt;` answer hint.**
  D10 names the docs it rewrites (`docs/overview.md` and the two package docs) but not `contracts/orchrole/role-slack.md`. Every orch reads that file as instruction, and after this change it describes the removed routing.
  The handler should weigh this against the rubric's exclusion of call-site enumeration. It is an agent-instruction reader whose meaning the decisions falsify, not a missing call site.
  Evidence: `contracts/orchrole/role-slack.md:4-6,22-23`, `decision-record.md:194,217,253`.
- **F7 — LOW, consistency. D10: existing refusals keep naming the removed `--channel` flag.**
  The relay's setup-missing refusals and `RequireConfigured`'s refusal offer `lym slack setup --channel &lt;id&gt; --operator &lt;user-id&gt;` as the way forward, and D10 removes that flag. Following that way forward would fail.
  The record's `PATTERN-refusal-way-forward` constraint covers only "each new refusal".
  Evidence: `internal/slackengine/inbound.go:228`, `internal/slackengine/outbound.go:81`, `internal/slackengine/credentials.go:87`, `decision-record.md:241,284`.

**Tool calls:**

1. A grep for `:SLACK` and `SlackProgram` senders outside the Slack packages, for readers of the old routing text in contracts and docs, and for the `Beams()` reader. My context did not show who else posts to the relay or which instructions describe its routing.
2. `internal/beamcli/dobbydeps.go:255-300`, `contracts/orchrole/role-slack.md`, and a grep for readers of the setup, cursor and presence paths. My context lacked the dobby reminder's sender form and the orch role text.
