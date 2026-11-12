```yaml
lens: existing-paths
outcome: reported
```

Task: run the existing-paths lens over the slack-machine-bridge decision record.

**Paths traced, per decision**

- D3, creel's per-hub outbound workers and scheduler start scan (`internal/creelengine/outbound.go` `pendingOutbound`, `wakeOutbound`): survives, because D3 says the scheduler skips these messages.
- D3/D2, the reply path for a held inbound message (`internal/creelengine/verbs.go:354-367`): **breaks** (F3).
- D2, the forward guard (`verbs.go:1035`): **breaks** (F3).
- D7, clearing of the relay's outbox record (`internal/slackengine/outbound.go:107`, `outboxRecord`): **breaks** (F1).
- D7, the existing relay's mailbox read (`postMessages`, `outboundRoute`): survives, because machine-outbound mail never reaches the SLACK mailbox.
- D8, the holder's route of an event no map holds (`internal/slackengine/holder.go:329-355`): changed by D8, and the change meets the existing resync hint (F2).
- D8, the existing resync hint (`holder.go:22`, `inbound.go` `readsForHint`): **breaks** (F2).
- D8, inbound `handle`, which skips bot messages (`inbound.go:600-603`): survives only if bridge reads do not reuse `handle`. D8's ordered rule implies a separate path.
- D8, the removed-holder sweep (`internal/creelengine/sweep.go:239`): survives only if machine-inbound mail sets `SendingHub`, and setting it breaks the reply path. Part of F3.
- D8, a carried record read by an older binary (`internal/creelengine/fold.go:625`): survives. The older binary logs and ignores the record, as D8 intends.
- D9, the stamp (`fold.go:102` `HubStamp`, keyed on `SendingHub`): the record names this site, but it is part of F3.
- D4, retirement (`internal/slackengine/keeper.go:399` `retire` over the orch channel map): survives as stated. The bridge map needs its own pass, and D4 names one. After the peer archives the channel, `postRecovered` meets `is_archived`. D4 then drops the channel and finds it again, and that works because `ArchivedName` renames the channel before archiving and so frees the name.
- D4/D8, the bridge map file under `.lym/slack/` against the relay's wake watch (`relay.go`: it watches the inbox and the forwarding file only): survives.
- D6, the approved-peer record beside `slack.json`: no existing path removes or rewrites the credential directory (`internal/slackcli`, `credentials.go`). Survives.

**Findings**

- **F1** — MEDIUM, design, D7 (restart safety of bridge posts) and D3 (re-settle at start).
  - The record keeps each accepted bridge post in "the relay's outbox record". The existing outbox's `Handled` means "handled and not yet acknowledged" for the SLACK mailbox read.
  - Every confirmed read wipes the whole record with `out.save(outboxRecord{})`. So the first orch-mail read after a bridge post erases that post's entry.
  - A restart then posts the message again, and D3's start-time re-settle finds nothing to settle.
  - The record never says the bridge entries live in their own record or field, or when they are cleared.
  - Evidence: `internal/slackengine/outbound.go:51-58` and `:106-108`.
- **F2** — MEDIUM, design, D8 (an event in an unmapped channel).
  - The existing `resync` hint makes a hub's relay read every open orch channel with a 24-hour thread scan (`inbound.go` `readsForHint`).
  - Today the holder acknowledges and drops an event in an unmapped channel (`holder.go:352-354`).
  - D8 turns every such event into a "resync hint" in every hub's inbox. Any message in any channel the bot was invited to would then set off a full orch-channel rescan and a users.conversations listing on every hub, at most once a second each.
  - The record does not separate the bridge resync from the existing hint, and states no bound on this edge.
  - Evidence: `internal/slackengine/holder.go:22`, `:329-355`; `internal/slackengine/inbound.go:358-365`.
- **F3** — MEDIUM, design, D2/D8/D9 (machine-inbound mail in creel's other-hub branches).
  - Every other-hub branch in creel keys on `Message.SendingHub`:
    - the stamp (`fold.go:103`);
    - the reply path, which journals `eventOutbound` with `ReceivingHub: original.SendingHub` and wakes the per-hub outbound worker (`verbs.go:354-367`);
    - the forward guard that keeps the message to strands of this hub (`verbs.go:1035`);
    - the sweep's undelivered return (`sweep.go:239`).
  - If machine-inbound mail sets `SendingHub`, `post --reply` is handed to a per-hub outbound worker for a "hub" that is a machine, which contradicts D3.
  - If it leaves `SendingHub` empty, the stamp is empty, the D2 forward restriction is skipped and the reply is journaled as a local post.
  - The record names only `HubStamp` and states neither behaviour.
  - Evidence: `internal/creelengine/fold.go:71-72`, `:102-115`; `internal/creelengine/verbs.go:354-367`, `:1035-1038`; `internal/creelengine/sweep.go:239`.

**Tool calls**

1. `fold.go` `HubStamp` and the carried-record fold: my context lacked how the stamp is chosen and how an older binary folds an unknown `of`.
2. `relay.go` watch and the keeper method index: my context lacked which files wake the relay and where retirement runs.
3. Removal paths in `slackcli`/`credentials.go` and the `sweep.go` holder check: my context lacked any path that rewrites the credential directory, and the sweep's other-hub handling.
4. Grep for `SendingHub` and the `eventInbound` fold: my context lacked which creel paths branch on other-hub origin.
5. `verbs.go` reply and forward branches and the `sweep.go` return: my context lacked the code those `SendingHub` branches run.
