## Goal

Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs

Takes: creel's Go program client, given a read-with-wait wrapper in creelcli and an exported pane-ancestry walk in creelengine; runproc, selfexec, internal/state and lymdirs as they stand
Delivers: a live per-hub relay started by prime bring-up, prime dobby ticks and anchor changes; the token file, the hub's setup file and the presence state under .lym/slack/; a SLACK program mailbox orchs and lym programs post to; the slack config module with remind_after_min; the orch-role rules, written on top of creel-cross-hub-mail's landed sender and trust rules; and the two PATTERN changes under PATTERN.md's live cluster ids, as the Sketch's Constraints allow: PATTERN-reap-children's double-fork exception goes into PATTERN-child-processes and its background file, and PATTERN-slack-boundary joins PATTERN-agent-addressing (messages only through creel) as a change to it, since that cluster guards agent messaging
Exports: slackengine's presence reader for a hub (terminal or slack), the remind_after_min config loader, and the SLACK program name and receiver helper other packages post to

## Scope

The part changes these packages:

- internal/slackengine (about 1600 changed lines)
- internal/slackcli (about 700 changed lines)
- internal/creelcli (about 60 changed lines)
- internal/creelengine (about 15 changed lines)
- internal/hubgeom (about 25 changed lines)
- internal/orchcli (about 30 changed lines)
- internal/orchengine (about 25 changed lines)
- internal/fabriccli (about 20 changed lines)
- internal/beamcli (about 20 changed lines)
- internal/githubclient (about 15 changed lines)
- internal/runproc (about 5 changed lines)
- internal/lymdirs (about 5 changed lines)
- internal/configreg (about 5 changed lines)
- cmd/lym (about 10 changed lines)
- contracts/orchrole (about 15 changed lines)
- pattern (about 30 changed lines)

## Decisions

### D1 — One private Slack channel per hub, one Slack app per machine

Each hub has its own Slack channel; one Slack app, with one bot token, serves every hub on the machine.
The relay reads only its hub's channel, so a message needs no hub name, and the channel's name tells the operator which hub speaks.
Rationale: a channel per hub keeps each hub's relay self-contained, as creel is per hub, and needs no machine-wide registry; one app means one install and at most one admin approval in the employer's workspace.

### D2 — The Slack token lives in a machine credential file; channel and operator in a per-hub file only setup writes

- The bot token (`xoxb-…`) is stored in a file `slack.json` in the same per-user lym directory as `internal/githubclient/cache.go`'s credential cache, written atomically with owner-only permissions, never in a repository, a config layer or `lym config env`.
- `lym slack login` reads the token from standard input, never from an argument, checks it with Slack's `auth.test`, and writes the file.
- `lym slack setup` checks the channel with `conversations.info` (the bot a member), the user with `users.info`, and only then writes the hub's channel id and the operator's Slack user id to a setup file under the hub's `.lym/slack/` directory through `internal/state`, with owner-only permissions.
  No config module holds the two ids, so no `lym config` write reaches them; a new `slack` config module holds only `remind_after_min` (D8).
- The bot scopes are `channels:history` and `channels:read` for a public channel or `groups:history` and `groups:read` for a private one, `users:read` and `chat:write`.
- The relay accepts a message only when its `user` equals the configured operator id.
  It reads the channel and operator id from the setup file at each poll, so a `lym slack setup` reaches a live relay at its next poll, and a changed channel takes D4's new-channel path at that poll.
- `lym slack login` and `lym slack setup` refuse when run from any live strand's pane, an orch's included, by D6's pane-ancestry check, its unknown-pane refusal included, naming the operator's terminal as the way forward.

Rationale: `lym config env` exports its variables into every agent pane, which would hand the token to every agent; the githubclient cache is the existing precedent for a machine-global credential outside any repository.
The two ids stay out of config because `lym config` guards a hub-wide module's machine layer only against a mistaken write (`internal/configcli/hubwide.go`, `hubWriteRefusal`), which lets the prime orch's pane write past setup's checks; with the ids in the setup file, every lym path to them runs setup's checks and its pane refusal.
Bound: any process of the operator's user can read the files, as it can read the GitHub token cache today; the token reaches no agent's environment or prompt.
Bound: only `lym slack login` and `lym slack setup` write the token, channel and operator id, and both refuse in every live strand's pane; the refusal guards a mistaken run, not a determined agent, so a process of the operator's user that writes the files directly, or that runs outside every pane, can still repoint them.
Bound: a channel is relayed by one machine; two machines configured with the same channel both forward each message, and the setup help text says to give each machine's hub its own channel.

### D3 — A per-hub relay process, `lym slack relay`

- `lym slack relay` is a long-lived process per hub with a single-instance lock, registered in `internal/runproc` under `runproc.HubOwner` with a role of its own, its spawn logged, its state and log under the hub's `.lym/slack/` directory named through `internal/lymdirs`.
- It is started through a double fork: the starter spawns a detached intermediate child in a new session, which starts the relay and exits at once, so the relay is reparented out of the starter's process tree; the starter waits on the intermediate, and the intermediate's release of the relay is the one exception `PATTERN-reap-children` admits (PATTERN changes); the relay waits until its parent is no longer that intermediate before its first creel read.
  Bound: the relay is never in a pane's process tree at its first read, whoever starts it, so creel reads it as no strand; this rests on no process in a pane's tree being a child subreaper, since an orphan is reparented to its nearest subreaper ancestor, and lym sets none.
- It is started when the hub's Slack setup is complete, the token file and the hub's setup file both present: by the prime orch's bring-up (`orchcli.BeamUp` for the prime), by `lym slack away` and `lym slack start`, by each tick of the prime's dobby while presence is `slack`, so a relay that died while the operator is away comes back without a terminal, and by a prime anchor change once it ends, whether it lands or fails and whatever dobbies it starts again, since the change's `EndDobbies` ends every `runproc.HubOwner` process but a push or a sync. A start that finds it live does nothing.
- `lym slack start` refuses when run from a live strand's pane other than an orch's, by D6's pane-ancestry check, its unknown-pane refusal included, naming the operator's terminal or an orch's pane.
- `lym slack stop` first sets presence to `terminal`, with D6's notices, and then ends the relay through `runproc`'s `EndRole`, so no dobby tick restarts it.
  Bound: a stopped relay stays down until a start, `lym slack away` or the next prime bring-up or anchor change.
- On a replaced lym binary it re-execs itself, as the hub watchdog does, calling `proc.ReapInheritedChildren` first and taking its lock again before its next poll.
  Bound: the lock is free from the exec until the new image takes it, so a start in that window can launch a second relay; whichever of the two fails to take the lock exits before its first poll, so only one relay polls and no message is forwarded twice.
- Its first act after the lock is a creel read as `<shortname>:SLACK`, which makes that program a known creel receiver; it comes before any Slack poll.
  Every refused or failed read is written to its log under `.lym/slack/`, the log `lym slack away` names.
  When creel refuses a first read, this run's or the first after a re-exec, because the relay runs in a non-orch strand's pane, the relay ends, releasing its lock; any other refusal or failure, as while a live strand's pane process is unknown or the daemon is down, is retried with the poll's backoff, the relay holding its lock and polling no Slack until a read succeeds.
  Bound: `lym slack relay` run by hand from a non-orch strand's pane polls Slack for nothing and leaves the lock free for the next start; a relay started while creel cannot place it waits, and comes up once creel can.
- `lym slack status` reports presence, whether the relay is live, whether a pause covers the prime orch, the channel and operator id from the setup file and the time of the last successful Slack call.
- With the token file or the setup file missing, the relay and `lym slack away` refuse, naming `lym slack login` and `lym slack setup`.

Rationale: the creel daemon decides only delivery and holds no transport, so the Slack transport stays outside it; one process per hub matches creel's per-hub geometry.

### D4 — Inbound: the relay polls the channel and posts the operator's text to creel as `<shortname>:SLACK`

- The relay reads the channel with `conversations.history` from a cursor, the channel id and the `ts` of the newest message it has handled there, persisted under `.lym/slack/` through `internal/state`; it pages until `has_more` is false before handling the messages oldest first.
  A poll that finds no cursor, or a cursor for a channel other than the configured one, first writes the cursor as the configured channel and the `ts` of its newest message, or the current time on an empty channel, and only then reads, so it forwards no history, of a channel relayed before included, and a later restart reads from there.
- Every message a poll returns is handled once, oldest first, and the cursor moves past it once its handling ends, whether it was forwarded, answered or skipped; a forwarded message's creel post comes before its cursor write.
  A creel post that fails or is refused, as creel refuses a program claim from outside every pane while a live strand's pane process is unknown, or while its daemon is down, leaves the cursor before that message, is logged at Warn and is retried with the poll's backoff, so later messages, presence commands included, wait behind it; after 5 minutes of failures the relay answers in Slack that the message was not delivered, quoting creel's refusal and asking for it to be sent again, and moves the cursor past it.
- It polls every 5 seconds, doubles the wait after each empty poll up to 60 seconds, and returns to its floor on an inbound message, an outbound post that succeeds or a presence change it reads at each poll.
  A `429` waits out Slack's `Retry-After` and doubles that relay's floor, up to 60 seconds, for the rest of its run, so more hubs than the shared limit fits degrade to slower polling.
- It takes a message only when it is top-level (no `thread_ts` other than its own `ts`), carries no `subtype` and no `bot_id`, and its `user` is the configured operator; every other message is skipped and logged at Debug.
- It unescapes Slack's `&amp;`, `&lt;` and `&gt;`, turns `<url|label>` and `<url>` into the URL when the bracketed value starts with `http://`, `https://` or `mailto:`, and leaves the rest of the text, a `<#C…|name>` channel or `<!here>` form and a `<@U…>` mention included, as Slack delivers it.
- A text that is exactly `away`, `back` or `status`, case and surrounding space ignored, is a presence command (D6) and goes to no orch.
- A text opening `@<slug> ` goes, without the prefix, to the orch of the beam `<slug>`, `<shortname>:<slug>:orch`; any other text goes to the prime orch, `<shortname>:orch`.
  A `<slug>` that names no beam of the hub is answered in Slack with the hub's beam slugs, and nothing is posted to creel.
- The relay posts through the creel Go client as the program `<shortname>:SLACK`, the body `[slack] <text>`.
  A body past creel's 4 KiB notice cap, the size a pane-mail orch still gets typed in full, is answered in Slack asking for a shorter message, and nothing is posted.
- When a pause covers the receiving orch, the relay still posts and answers in Slack that the hub is paused and the message waits for the resume.

Rationale: polling needs only the bot token and no inbound connection or WebSocket library, works behind the corporate network, and stays inside the history method's limit for an internal app (Constraints).
Bound: a crash between the creel post and the cursor write forwards that message once more; a refused creel post holds later messages, presence commands included, for at most 5 minutes, and is then answered in Slack as not delivered; a thread reply is never read; a hold on the orch delays the mail, with no Slack answer, for as long as the holder keeps renewing it, at most an hour past its last renewal; a message routed to a beam torn down before its orch reads it stays unread in creel and is not answered in Slack, and `lym creel log --to <shortname>:<slug>:orch` shows it.

### D5 — Outbound: orch and program mail to `<shortname>:SLACK` is posted to the channel

- The relay reads its own program mailbox with `read --wait` and posts each message with `chat.postMessage`, oldest first, as `[<sender>] <body>`, with `&`, `<` and `>` escaped.
- A message whose stamped sender carries a beam slug, `<shortname>:<slug>:orch` or `<shortname>:<slug>:<PROGRAM>`, ends with `(answer: @<slug> …)`, so the operator's answer routes back to that beam's orch (D4).
- It posts only mail whose stamped sender is an orch strand or a program of the hub other than `<shortname>:EXTERNAL`; mail from any other strand, and EXTERNAL mail, is dropped and logged at Warn.
- At a read it skips, without posting, each message creel stamped more than 24 hours earlier, recording it in the record below so the read's acknowledgement covers it, and posts one line naming how many it skipped and `lym creel log --to <shortname>:SLACK`.
- It keeps a record, under `.lym/slack/` through `internal/state`, of the creel ids of the current read already posted or skipped.
  A read's handler stops posting before creel's 30-second acknowledgement deadline (`ackReadTimeout` in `internal/creelengine/server.go`) and releases the read with the rest unposted; when a post fails it releases the read unacknowledged at once.
  Either way it waits outside the handler, with the poll's backoff or a `429`'s `Retry-After`, and reads again, skipping the ids the record holds.
  It acknowledges the read once every id is posted or skipped, and clears the record only after the daemon confirms the acknowledgement; a refused or lost acknowledgement keeps the record.
  After Slack refuses one message 5 times with an `ok: false` answer other than `ratelimited`, such as `msg_too_long`, it posts one line naming the message's id and sender and `lym creel log --to <shortname>:SLACK`, and counts the message as handled, whether that line posts or not; a failed transport, a `429` or a `5xx` counts toward no cap and is retried, a `429` after its `Retry-After` and the rest with the backoff.
- A reply to a `<shortname>:SLACK` message (`lym creel post --reply <id>`) reaches the relay's mailbox, so the orch answers Slack mail the way it answers any mail.

Rationale: every agent's and program's outbound message goes through creel's journal, so no agent gets a Slack tool of its own, and mail posted while the relay is down waits in creel and is posted when it is back; only the relay's own answers to the operator's Slack messages are posted directly.
Bound: delivery is at least once, a crash between a post and its record write posting that message once more; a refused or lost acknowledgement reposts nothing the record holds; mail waits in creel for a relay that stays down with no limit, since retention carries unread mail and a program's read forward (`internal/creelengine/journal.go` `retain`), and mail older than 24 hours at the read is not posted; only orchs and lym programs reach the channel, each post named by the sender creel stamped.

### D6 — Presence state per hub

- Presence is `terminal` or `slack`, stored under `.lym/slack/` through `internal/state`; a hub with no stored state is `terminal`.
- `lym slack away` starts the relay when it is down, waits until the relay has made its first read, and only then sets `slack`; it refuses, naming the relay log, when the relay has not made its first read within 30 seconds, and names creel's refusal when creel refused that read, as it does while a live strand's pane process is unknown (D4).
- `lym slack back` sets `terminal`, and `lym slack status` prints the state with D3's report; the Slack texts `away`, `back` and `status` do the same, each answered in Slack with the resulting state.
- `lym slack away`, `lym slack back` and `lym slack stop` refuse when run from a live strand's pane other than an orch's, read from the pane ancestry against the hub's strands as creel's sender resolution does; `away` and `back` name the operator's terminal or the Slack texts `away` and `back` as the way forward, and `stop` names the operator's terminal or an orch's pane.
  This pane-ancestry check, which D2 and D3 use too, also refuses, as creel's resolution does, a caller in no known pane while any live strand's pane process is unknown or the hub's strands cannot be read, since the caller could be a strand; that refusal names what is unknown and, as the way forward, waiting until it is known or the strand ends and then running the verb again.
- A change mails a notice from `<shortname>:SLACK` to the prime orch and to the orch of every beam of the hub, its body opening `[slack presence]` and naming the new state; it asks for no reply.
- The orch's resume pointer (`lym orch resume-context` and the typed pointer) carries one line naming the presence while it is `slack`, so an orch that cycled its context still knows.

Rationale: the orch learns the state from mail and from its resume pointer, with no polling of its own (orch role run-loop rule 4).
Bound: presence is changed only by the operator, an orch, a process outside every pane, and the relay on the configured operator's Slack text; no process inside a non-orch strand's pane changes it, but a strand that deliberately starts a process outside its pane is not stopped, as D7's bound states for creel.
Bound: while a live strand's pane process stays unknown, `away`, `back` and `stop` refuse outside every pane, as creel then refuses program mail from outside every pane; the Slack texts `away`, `back` and `status` still set or print the presence, since the relay reads them from Slack, and its notices wait on creel.

### D7 — The orch role's Slack rules

`contracts/orchrole/orch-role.md` gains these rules:

- Mail from `<shortname>:SLACK` whose body opens `[slack]` carries the operator's own words, with the same authority as a line typed in the orch's pane; the orch answers it with `lym creel post --reply <id>`, which reaches Slack.
- Mail opening `[slack presence]` tells the orch the new presence and is not answered.
- While presence is `slack`, every message the orch means for the operator, a design question, a go-ahead it needs, a status report, a relayed owner halt and an answer to the operator, is also posted to `<shortname>:SLACK` with `lym creel post`, leaving out the attach command, which needs a terminal.
- While presence is `terminal`, the orch posts to `<shortname>:SLACK` only replies to mail that came from it.

Rationale: what is meant for the operator is the orch's judgment, so the orch mirrors it; Go mirrors nothing from the pane.
Bound: the authority rests on creel's program trust, the same as `LOOM` and `DOBBY` mail: a program name is taken only from a process in no pane or in an orch pane, inside the user's 0700 lymtemp root, so any process of the operator's user outside a pane can post as `SLACK`; the relay itself forwards only the configured operator's text.

### D9 — The `lym slack` CLI module

`lym slack` is a cobra subtree mounted in `cmd/lym/main.go` with the verbs `login`, `setup`, `start`, `stop`, `relay`, `away`, `back` and `status`; `relay` is hidden with the `internal` audience and carries no operator audience, since only the starter's double fork runs it, a run by hand being bounded by D3's refused first read; the rest are operator verbs, and `away`, `back` and `status` are also in the orch's command index.

### Added after Discussion (parent, 2026-11-05): Child-process rule folds into PATTERN-child-processes

Decision: The double-fork exception goes into the live cluster PATTERN-child-processes and its background file, in place of the declared PATTERN-reap-children, which is withdrawn: no top-level PATTERN-reap-children entry is added or changed.

Rationale: The Sketch's Delivers line and Constraints fold both declared ids into the live cluster ids since pattern-distill landed; a new top-level entry would undo that fold.

### Added after Discussion (parent, 2026-11-05): Slack boundary folds into PATTERN-agent-addressing

Decision: The Slack boundary joins the live cluster PATTERN-agent-addressing as a change to it, in place of the declared PATTERN-slack-boundary, which is withdrawn: no top-level PATTERN-slack-boundary entry is added.

Rationale: PATTERN-agent-addressing already guards agent messaging (messages only through creel), which the Sketch names as the cluster the Slack rule joins.

### Added after Discussion (parent, 2026-11-05): Inbound poll: a Hub.Paused error after an accepted post still moves the cursor

Decision: Take F3's fix as written. In card 07 step 3 ('After a successful post'): a Hub.Paused error is logged at Warn once per episode, sends no paused answer, and counts the message as handled so the cursor moves past it. A Hub.Beams error for a text opening with an at-sign slug happens before any post, so the poll returns it with the cursor left before the message and retries with the backoff. TestInboundPoll gains one row: Paused errors after an accepted post; the message is forwarded once, the error logged, no answer sent, the cursor moved, and a second poll forwards nothing.

Rationale: The post already reached creel; retrying would forward the operator's message twice, which is worse than a missing paused answer. A Beams error comes before any post, so retrying it duplicates nothing.

### Added after Discussion (arbiter, 2026-11-05): Plan review round 3: the relay restart cap's lifetime rule answers F1

Decision: Card 09 (slackengine-relay) keeps its round-3 restart cap as written: starter.ensure judges each spawned relay by its lifetime, LastSlackOK minus LastSpawnAt, against relayHealthyLifetime (10 minutes); a lifetime under it counts one failure, five in a row answer gave-up, and step 4 no longer resets StartFailures. Restarts of relays that each lived relayHealthyLifetime open a new count and stay bounded in rate, at most one per relayHealthyLifetime.

Rationale: Round 3's only design-wrong finding, F1 (relay-restart-cap-unreachable), said a relay dying after its first read respawned once a minute with no cap. The fixer applied F1's own proposed fix, and card 09 now counts that death as a failure and has a TestEnsureRelay row where five such deaths end in gave-up, so the spawn count of quick failures is capped as PATTERN-child-processes asks. The rate-bounded restart of long-lived relays is the reset F1's fix itself prescribed.

## Constraints

- Slack facts (docs.slack.dev, fetched 2026-11-05):
  - `conversations.history` returns top-level messages; a thread reply is returned only from `conversations.replies`. A message's `thread_ts` equals its own `ts` on a thread parent and differs on a reply (docs.slack.dev/messaging/retrieving-messages).
  - `conversations.history`'s `oldest` is exclusive unless `inclusive` is true; messages come newest first, with `has_more` and a `response_metadata.next_cursor` to page; a `429` carries a `Retry-After` header in seconds (docs.slack.dev/reference/methods/conversations.history).
  - Slack sends a URL in a message's text as `<url>` or `<url|label>` and a user mention as `<@U…>` (docs.slack.dev/messaging/formatting-message-text).
  - The message event carries `user` for the author, and `bot_id` or subtype `bot_message` on messages an integration posted (docs.slack.dev/reference/events/message).
  - The 2025-05-29 rate-limit change (1 request a minute, 15 messages) applies to commercially distributed apps outside the Marketplace; "internal customer-built applications are not impacted" and keep Tier 3, 50+ requests a minute (docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps). The limit is per app per workspace, so every hub's relay on the machine's one app (D1) shares it; D4's 5-second floor is 12 requests a minute per hub, and four hubs fit at 48 requests a minute.
- `go.mod` carries no Slack or WebSocket library today; the relay calls the Web API over `net/http`, adding no dependency.
- The relay's loops join the declared list in `cmd/lym/daemonwakeups_test.go`.
- The PATTERN IDs this record names are read under their cluster IDs if `pattern-distill` lands first; `PATTERN-slack-boundary` then joins the cluster that guards agent messaging, as a change to it, when one exists at Plan time.

## Auto-mode assumptions

- One channel per hub (D1), polling (D4), the `@<slug>` routing prefix (D4), and reminders for PR-Review and Sampler-Session every 30 minutes (D8) are auto-picks; no operator answered.

## Open risks

- The employer's Slack workspace may require admin approval of every app ("Only allow pre-approved apps", slack.com/help/articles/222386767); the relay cannot run until the operator has the app installed. This blocks no part of the build.

## Acceptance criteria

- D1, D4: with a fake Slack API, a top-level message from the configured operator in the hub's channel reaches the prime orch's creel mailbox as a `[slack]` message from `<shortname>:SLACK`; a message from another user, a bot message, a thread reply and a message with a subtype reach no mailbox.
- D4: a message opening `@<slug> ` for a beam of the hub reaches `<shortname>:<slug>:orch` without the prefix; one for an unknown slug gets one Slack answer listing the beams and nothing in creel.
- D4: the relay forwards no message older than its first start; across polls and a restart after the cursor write, each message, forwarded, answered or skipped, is handled exactly once; a message written while a relay that never handled a message is down is forwarded once after its restart; a body past 4 KiB gets one Slack answer and no creel post; a message whose creel post creel refuses is forwarded once creel accepts it within 5 minutes, and is otherwise answered once in Slack as not delivered, never skipped silently; after a `lym slack setup` changes the channel, to a new one or back to one relayed before, the relay forwards none of that channel's history.
- D4: a history answer with `has_more` is paged to its end and handled oldest first; `<https://…|label>` and `<https://…>` reach the orch as the URL, and `<#C…|name>` and `<!here>` reach it unchanged.
- D4: the poll wait doubles after empty polls up to 60 seconds, returns to the floor on activity, and a `429` waits the `Retry-After` it names and doubles the floor.
- D4: a message for an orch a pause covers is posted to creel and answered in Slack that the hub is paused.
- D5: mail from an orch or a program to `<shortname>:SLACK`, a reply to a `[slack]` message included, is posted as `[<sender>] <body>` with `&`, `<` and `>` escaped; a post whose sender carries a beam slug, a beam orch or a beam program, carries the `@<slug>` answer hint; mail from any other strand, and mail from `<shortname>:EXTERNAL`, is not posted.
- D5: a Slack failure on the second message of a read, then success, posts each message exactly once; a message Slack refuses 5 times with `ok: false` is replaced by one line naming its id, its sender and `lym creel log --to <shortname>:SLACK`, and the read is acknowledged; a transport failure keeps the message queued past 5 tries; a backlog whose posting outlasts the acknowledgement window posts each message exactly once.
- D5: mail stamped more than 24 hours before the read is acknowledged without a post, and one line names how many were skipped and `lym creel log --to <shortname>:SLACK`.
- D2: `lym slack login` reads the token from standard input, writes the credential file with owner-only permissions after `auth.test` answers ok, and refuses a token Slack rejects; no config layer or env file holds the token; `lym slack login` and `lym slack setup` run from any strand's pane, an orch's included, or from outside every pane while a live strand's pane process is unknown, refuse and write nothing; the channel and operator id live only in the setup file, so a `lym config slack` write, from a pane or not, does not repoint a live relay.
- D2, D3: the relay and `lym slack away` refuse when the token file or the setup file is missing, naming `lym slack login` or `lym slack setup`.
- D3: a second relay start for the same hub starts nothing; `lym slack start` run from a non-orch strand's pane, or from outside every pane while a live strand's pane process is unknown, refuses and starts nothing; a `lym slack relay` run from a non-orch strand's pane exits after creel refuses its first read, forwards nothing, and a later start takes the lock; a relay whose first read creel refuses while a live strand's pane process is unknown keeps its lock, polls no Slack, and makes its first read once the pane process is known; a relay start called from inside a non-orch strand's pane brings up a relay whose first read creel accepts as `<shortname>:SLACK`; `lym slack stop` with presence `slack` sets `terminal` and ends the relay, and the next prime dobby tick starts nothing; a prime dobby tick with presence `slack` starts a relay that is down; the relay is live after a prime anchor change with presence `terminal`, whether the change lands or fails.
- D6: `lym slack away` sets `slack` only after the relay's first read and refuses when the relay does not come up; `lym slack back` and `lym slack status` set or print the presence; the Slack texts `away`, `back` and `status` set or print it and are answered in Slack; a change mails every orch of the hub a `[slack presence]` notice; a hub with no presence file reads `terminal`.
- D6: `lym slack away`, `back` and `stop` run from a non-orch strand's pane, or from outside every pane while a live strand's pane process is unknown, refuse and leave the presence and the relay unchanged; run from an orch's pane, or outside every pane while every live strand's pane process is known, they take effect.
- D6: the orch resume pointer carries the presence line while presence is `slack` and none while `terminal`.
- D9: `lym slack` is mounted under the root with every verb carrying a `Short` and an audience, as `cmd/lym/clitree_test.go` checks.
- Review: D7: `contracts/orchrole/orch-role.md` says that `[slack]` mail from `<shortname>:SLACK` carries the operator's authority, that `[slack presence]` mail is not answered, that the orch mirrors every operator message to `<shortname>:SLACK` without the attach command while presence is `slack`, and posts only replies while it is `terminal`.
- Review: D2: the `lym slack login` and `lym slack setup` help text names the bot scopes of D2, inviting the bot to the channel, finding the operator's Slack user id, and one channel per machine.

## PATTERN changes

- `PATTERN-child-processes` — changes: Starting or ending a process: log the spawn via `internal/logger`; its starter waits on it, bar a double fork's intermediate, which exits at once; a run's processes register in `internal/runproc` and end only through it. — added after Discussion (parent, 2026-11-05)
- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; Slack only via `internal/slackengine`, its token only in its machine credential file; operator Slack text reaches agents only as creel mail from `<shortname>:SLACK`. — added after Discussion (parent, 2026-11-05)
