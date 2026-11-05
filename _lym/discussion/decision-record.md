# Decision record — reed-slack-relay

## Goal

The operator follows and steers a hub's orchs from the phone through Slack, since the work machine's Claude account cannot be reached from the phone.
A Slack message from the operator reaches an orch as creel mail with the operator's authority; the orch's replies, reports and lym's operator-wait reminders reach the operator's Slack channel.
The orchs and lym know whether the operator is at the terminal or reachable only through Slack.

## Scope

In:

- A new `slack` module: `internal/slackengine` (new) for the Slack Web API, presence and relay logic, and `internal/slackcli` (new) for the `lym slack` verbs.
- A per-hub relay process that reads the hub's Slack channel and forwards the operator's messages into creel, and posts the mail creel holds for it to the channel.
- Credentials and per-hub settings for the relay, kept out of every repository.
- The presence state, its verbs and its Slack commands.
- Operator-wait reminders posted to Slack while presence is `slack`.
- The orch role's rules for Slack mail and for mirroring operator messages to Slack.
- One new PATTERN entry for the Slack boundary.

Out:

- Slack threads, files, reactions, slash commands and interactive components: the relay reads and writes top-level plain-text messages only.
- Creel mail between hubs (`creel-cross-hub-mail`): each hub runs its own relay against its own channel.
- Creating the Slack app, installing it in the workspace and getting a workspace admin's approval: the operator's own steps, which the `lym slack` help text describes.
- A presence change made automatically from operator activity in a pane.

## Decisions

### D1 — One private Slack channel per hub, one Slack app per machine

Each hub has its own Slack channel; one Slack app, with one bot token, serves every hub on the machine.
The relay reads only its hub's channel, so a message needs no hub name, and the channel's name tells the operator which hub speaks.
Rationale: a channel per hub keeps each hub's relay self-contained, as creel is per hub, and needs no machine-wide registry; one app means one install and at most one admin approval in the employer's workspace.

### D2 — The Slack token lives in a machine credential file; channel and operator in machine config

- The bot token (`xoxb-…`) is stored in a file `slack.json` in the same per-user lym directory as `internal/githubclient/cache.go`'s credential cache, written atomically with owner-only permissions, never in a repository, a config layer or `lym config env`.
- `lym slack login` reads the token from standard input, never from an argument, checks it with Slack's `auth.test`, and writes the file.
- A new `slack` config module holds, as machine-layer keys, the hub's channel id and the operator's Slack user id; `lym slack setup` checks the channel with `conversations.info` (the bot a member), the user with `users.info`, and only then writes them.
- The bot scopes are `channels:history` and `channels:read` for a public channel or `groups:history` and `groups:read` for a private one, `users:read` and `chat:write`.
- The relay accepts a message only when its `user` equals the configured operator id.

Rationale: `lym config env` exports its variables into every agent pane, which would hand the token to every agent; the githubclient cache is the existing precedent for a machine-global credential outside any repository.
Bound: any process of the operator's user can read the file, as it can read the GitHub token cache today; the token reaches no agent's environment or prompt.
Bound: a channel is relayed by one machine; two machines configured with the same channel both forward each message, and the setup help text says to give each machine's hub its own channel.

### D3 — A per-hub relay process, `lym slack relay`

- `lym slack relay` is a long-lived process per hub with a single-instance lock, registered in `internal/runproc` under `runproc.HubOwner` with a role of its own, its spawn logged, its state and log under the hub's `.lym/slack/` directory named through `internal/lymdirs`.
- It is started detached in a new session, as the creel daemon is, so it sits outside every pane's process tree whoever starts it and creel reads it as no strand.
- It is started when the hub's `slack` config is complete: by the prime orch's bring-up (`orchcli.BeamUp` for the prime), by `lym slack away` and `lym slack start`, and by each pass of the prime's dobby while presence is `slack`, so a relay that died while the operator is away comes back without a terminal. A start that finds it live does nothing.
- `lym slack stop` ends it through `runproc`'s `EndRole`.
- On a replaced lym binary it re-execs itself, as the hub watchdog does, calling `proc.ReapInheritedChildren` first and taking its lock again.
- Its first act after the lock is a creel read as `<shortname>:SLACK`, which makes that program a known creel receiver.
- `lym slack status` reports presence, whether the relay is live, whether a pause covers the prime orch, the channel and the time of the last successful Slack call.
- With the `slack` config incomplete, the relay and `lym slack away` refuse, naming `lym slack login` and `lym slack setup`.

Rationale: the creel daemon decides only delivery and holds no transport, so the Slack transport stays outside it; one process per hub matches creel's per-hub geometry.

### D4 — Inbound: the relay polls the channel and posts the operator's text to creel as `<shortname>:SLACK`

- The relay reads the channel with `conversations.history` from a cursor, the `ts` of the newest message it has handled, persisted under `.lym/slack/` through `internal/state`; a first start with no cursor starts from the current time and forwards no history.
- Every message a poll returns is handled once, oldest first, and the cursor moves past it once its handling ends, whether it was forwarded, answered or skipped; a forwarded message's creel post comes before its cursor write.
- It polls every 5 seconds, doubles the wait after each empty poll up to 60 seconds, and returns to 5 seconds on an inbound message, an outbound post or a presence change it reads at each poll; a `429` waits out Slack's `Retry-After`.
- It takes a message only when it is top-level (no `thread_ts` other than its own `ts`), carries no `subtype` and no `bot_id`, and its `user` is the configured operator; every other message is skipped and logged at Debug.
- It unescapes Slack's `&amp;`, `&lt;` and `&gt;`, and leaves the rest of the text as the operator wrote it.
- A text that is exactly `away`, `back` or `status`, case and surrounding space ignored, is a presence command (D6) and goes to no orch.
- A text opening `@<slug> ` goes, without the prefix, to the orch of the beam `<slug>`, `<shortname>:<slug>:orch`; any other text goes to the prime orch, `<shortname>:orch`.
  A `<slug>` that names no beam of the hub is answered in Slack with the hub's beam slugs, and nothing is posted to creel.
- The relay posts through the creel Go client as the program `<shortname>:SLACK`, the body `[slack] <text>`.
  A body past creel's 4 KiB notice cap, the size a pane-mail orch still gets typed in full, is answered in Slack asking for a shorter message, and nothing is posted.
- When a pause covers the receiving orch, the relay still posts and answers in Slack that the hub is paused and the message waits for the resume.

Rationale: polling needs only the bot token and no inbound connection or WebSocket library, works behind the corporate network, and stays inside the history method's limit for an internal app across the machine's hubs (Constraints).
Bound: a crash between the creel post and the cursor write forwards that message once more; a thread reply is never read; a hold on the orch delays the mail up to the hold's hour with no Slack answer.

### D5 — Outbound: orch and program mail to `<shortname>:SLACK` is posted to the channel

- The relay reads its own program mailbox with `read --wait` and posts each message with `chat.postMessage`, oldest first, as `[<sender>] <body>`, with `&`, `<` and `>` escaped.
- A message from a beam's orch, `<shortname>:<slug>:orch`, ends with `(answer: @<slug> …)`, so the operator's answer routes back to that orch (D4).
- It posts only mail whose stamped sender is an orch strand or a program of the hub; mail from any other strand is dropped and logged at Warn.
- It acknowledges a read only after every message in it was posted; when a post fails it releases the read unacknowledged, waits with the poll's backoff and reads again, so the messages posted before the failure are posted once more.
- A reply to a `<shortname>:SLACK` message (`lym creel post --reply <id>`) reaches the relay's mailbox, so the orch answers Slack mail the way it answers any mail.

Rationale: every outbound path goes through creel's journal, so no agent gets a Slack tool of its own, and mail posted while the relay is down waits in creel and is posted when it is back.
Bound: delivery is at least once; creel's seven-day retention bounds how long mail waits for a relay that stays down; only orchs and lym programs reach the channel, each post named by the sender creel stamped.

### D6 — Presence state per hub

- Presence is `terminal` or `slack`, stored under `.lym/slack/` through `internal/state`; a hub with no stored state is `terminal`.
- `lym slack away` starts the relay when it is down, waits until the relay has made its first read, and only then sets `slack`; it refuses, naming the relay log, when the relay is not up within 30 seconds.
- `lym slack back` sets `terminal`, and `lym slack status` prints the state with D3's report; the Slack texts `away`, `back` and `status` do the same, each answered in Slack with the resulting state.
- A change mails a notice from `<shortname>:SLACK` to the prime orch and to the orch of every beam of the hub, its body opening `[slack presence]` and naming the new state; it asks for no reply.
- The orch's resume pointer (`lym orch resume-context` and the typed pointer) carries one line naming the presence while it is `slack`, so an orch that cycled its context still knows.

Rationale: the orch learns the state from mail and from its resume pointer, with no polling of its own (orch role run-loop rule 4).

### D7 — The orch role's Slack rules

`contracts/orchrole/orch-role.md` gains these rules:

- Mail from `<shortname>:SLACK` whose body opens `[slack]` carries the operator's own words, with the same authority as a line typed in the orch's pane; the orch answers it with `lym creel post --reply <id>`, which reaches Slack.
- Mail opening `[slack presence]` tells the orch the new presence and is not answered.
- While presence is `slack`, every message the orch means for the operator, a design question, a go-ahead it needs, a status report, a relayed owner halt and an answer to the operator, is also posted to `<shortname>:SLACK` with `lym creel post`, leaving out the attach command, which needs a terminal.
- While presence is `terminal`, the orch posts to `<shortname>:SLACK` only replies to mail that came from it.

Rationale: what is meant for the operator is the orch's judgment, so the orch mirrors it; Go mirrors nothing from the pane.
Bound: the authority rests on creel's program trust, the same as `LOOM` and `DOBBY` mail: a program name is taken only from a process in no pane or in an orch pane, inside the user's 0700 lymtemp root, so any process of the operator's user outside a pane can post as `SLACK`; the relay itself forwards only the configured operator's text.

### D8 — Operator-wait reminders from dobby while presence is `slack`

- Each worktree's dobby, the prime's included, checks once a minute, while presence is `slack`, each running child whose current row is PR-Review or Sampler-Session.
- The wait's start is the `At` of the child's status history entry that made that row current.
- When the child has waited at that row for the reminder interval, the `slack` config key `remind_after_min` with a default of 30, dobby posts to `<shortname>:SLACK` one line naming the slug, the row and the minutes waited, and posts again each further interval while the wait lasts.
- A post creel refuses is logged at Warn and tried at the next check.
- Beam-Await, though in `loomshed.OperatorWaitRows`, is not reminded, since it waits on the beam's members as well as the operator.
- Reminders stop when presence goes back to `terminal`; on a return to `slack` the next check reminds of a wait already past the interval at once.

Rationale: dobby already reads each running child's status on its passes, and reminding is mechanical work, which dobby owns (`dormant-orch`).
Bound: the last-reminder time is kept in memory, so a restarted dobby may remind once early; a resume that re-enters the row restarts the wait's clock.

### D9 — The `lym slack` CLI module

`lym slack` is a cobra subtree mounted in `cmd/lym/main.go` with the verbs `login`, `setup`, `start`, `stop`, `relay`, `away`, `back` and `status`; `relay` is hidden and internal, the rest are operator verbs, and `away`, `back` and `status` are also in the orch's command index.

## Constraints

- Slack facts (docs.slack.dev, fetched 2026-11-05):
  - `conversations.history` returns top-level messages; a thread reply is returned only from `conversations.replies`. A message's `thread_ts` equals its own `ts` on a thread parent and differs on a reply (docs.slack.dev/messaging/retrieving-messages).
  - The message event carries `user` for the author, and `bot_id` or subtype `bot_message` on messages an integration posted (docs.slack.dev/reference/events/message).
  - The 2025-05-29 rate-limit change (1 request a minute, 15 messages) applies to commercially distributed apps outside the Marketplace; "internal customer-built applications are not impacted" and keep Tier 3, 50+ requests a minute (docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps). D4's 5-second floor is 12 requests a minute per hub.
- `go.mod` carries no Slack or WebSocket library today; the relay calls the Web API over `net/http`, adding no dependency.
- The relay's loops join the declared list in `cmd/lym/daemonwakeups_test.go`.
- The PATTERN IDs this record names are read under their cluster IDs if `pattern-distill` lands first; `PATTERN-slack-boundary` then joins the cluster that guards agent messaging, as a change to it, when one exists at Plan time.

## Auto-mode assumptions

- One channel per hub (D1), polling (D4), the `@<slug>` routing prefix (D4), and reminders for PR-Review and Sampler-Session every 30 minutes (D8) are auto-picks; no operator answered.

## Open risks

- The employer's Slack workspace may require admin approval of every app ("Only allow pre-approved apps", slack.com/help/articles/222386767); the relay cannot run until the operator has the app installed. This blocks no part of the build.
- `pr-review-wait-silent` asks for D8's PR-Review reminder; once this task lands, the parent should mark that note covered or remove it.
- A thread reply from the phone is never read (D4 bound), so an operator who replies in a thread loses that message.

## Acceptance criteria

- D1, D4: with a fake Slack API, a top-level message from the configured operator in the hub's channel reaches the prime orch's creel mailbox as a `[slack]` message from `<shortname>:SLACK`; a message from another user, a bot message, a thread reply and a message with a subtype reach no mailbox.
- D4: a message opening `@<slug> ` for a beam of the hub reaches `<shortname>:<slug>:orch` without the prefix; one for an unknown slug gets one Slack answer listing the beams and nothing in creel.
- D4: the relay forwards no message older than its first start; across polls and a restart after the cursor write, each message, forwarded, answered or skipped, is handled exactly once; a body past 4 KiB gets one Slack answer and no creel post.
- D4: the poll wait doubles after empty polls up to 60 seconds, returns to 5 seconds on activity, and a `429` waits the `Retry-After` it names.
- D4: a message for an orch a pause covers is posted to creel and answered in Slack that the hub is paused.
- D5: mail from an orch or a program to `<shortname>:SLACK`, a reply to a `[slack]` message included, is posted as `[<sender>] <body>` with `&`, `<` and `>` escaped; a beam orch's post carries the `@<slug>` answer hint; mail from any other strand is not posted.
- D5: a Slack failure on the second message of a read leaves the read unacknowledged, and a later read posts both messages.
- D2: `lym slack login` reads the token from standard input, writes the credential file with owner-only permissions after `auth.test` answers ok, and refuses a token Slack rejects; no config layer or env file holds the token.
- D2, D3: the relay and `lym slack away` refuse when the token, channel or operator id is missing, naming `lym slack login` or `lym slack setup`.
- D3: a second relay start for the same hub starts nothing; the relay runs in no pane's process tree, so creel takes its `SLACK` claim when `lym slack start` runs from a strand's pane; `lym slack stop` ends it; a prime dobby pass with presence `slack` starts a relay that is down.
- D6: `lym slack away` sets `slack` only after the relay's first read and refuses when the relay does not come up; `back`, `status` and the Slack texts `away`, `back` and `status` set or print the presence and answer in Slack; a change mails every orch of the hub a `[slack presence]` notice; a hub with no presence file reads `terminal`.
- D6: the orch resume pointer carries the presence line while presence is `slack` and none while `terminal`.
- D8: with presence `slack`, a child at PR-Review or Sampler-Session for the reminder interval, counted from the history entry that made the row current, gets one reminder per interval to `<shortname>:SLACK`; a child at Beam-Await, and any child while presence is `terminal`, gets none.
- D9: `lym slack` is mounted under the root with every verb carrying a `Short` and an audience, as `cmd/lym/clitree_test.go` checks.
- Review: D7: `contracts/orchrole/orch-role.md` says that `[slack]` mail from `<shortname>:SLACK` carries the operator's authority, that `[slack presence]` mail is not answered, that the orch mirrors every operator message to `<shortname>:SLACK` without the attach command while presence is `slack`, and posts only replies while it is `terminal`.
- Review: D2: the `lym slack login` and `lym slack setup` help text names the bot scopes of D2, inviting the bot to the channel, finding the operator's Slack user id, and one channel per machine.

## PATTERN changes

- `PATTERN-slack-boundary` — adds: Calling Slack: only `internal/slackengine` (new), with the token only from its machine credential file, never a config layer, env or repository; operator Slack text reaches agents only as creel mail from `<shortname>:SLACK`.
