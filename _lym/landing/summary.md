# Add a per-hub Slack relay and the lym slack verbs

Each hub gets a Slack relay, so the operator can talk to the hub's orchs from a private Slack channel while away from the terminal.
A new `internal/slackengine` holds the relay, and a new `internal/slackcli` mounts `lym slack` with the verbs `login`, `setup`, `start`, `stop`, `relay`, `away`, `back` and `status`.
The relay needs no inbound connection and no new dependency: it polls the Slack Web API over `net/http`.

## What changes

- Inbound: the relay polls the hub's channel with `conversations.history`.
  A top-level message from the configured operator reaches the prime orch's creel mailbox as `[slack] <text>` from `<shortname>:SLACK`.
  A text opening `@<slug> ` goes to that beam's orch instead.
  The texts `away`, `back` and `status` are presence commands and reach no orch.
- Outbound: mail from an orch or a lym program to `<shortname>:SLACK` is posted to the channel as `[<sender>] <body>`.
  Mail from other strands and from `<shortname>:EXTERNAL` is dropped and logged.
- Presence is `terminal` or `slack`, stored under `.lym/slack/`.
  A change mails a `[slack presence]` notice to every orch of the hub, and the orch's resume pointer names the presence while it is `slack`.
- Start-up: the relay starts at the prime orch's bring-up, on each prime dobby tick while presence is `slack`, at the end of a prime anchor change, and through `lym slack away` and `lym slack start`.
- `contracts/orchrole/orch-role.md` gains the Slack rules: `[slack]` mail carries the operator's authority, `[slack presence]` mail is not answered, and the orch mirrors what it means for the operator to Slack while presence is `slack`.
- A `slack` config module holds `remind_after_min`.
- `PATTERN-child-processes` gains the double-fork exception, and `PATTERN-agent-addressing` gains the Slack boundary.
- Supporting changes: `creelcli` gets a read-with-wait wrapper and a program-claim refusal inside a strand's pane, `creelengine` exports its pane-ancestry walk and orch test, and `hubgeom`, `lymdirs`, `runproc`, `githubclient`, `orchcli`, `orchengine`, `fabriccli` and `beamcli` get the small hooks the relay needs.

## Design decisions

- One private channel per hub and one Slack app per machine.
  A channel is relayed by one machine only; two machines on the same channel each forward every message.
- The bot token lives in `slack.json` in the per-user lym directory beside the githubclient cache, never in a repository, a config layer or `lym config env`.
  The channel and operator ids live in a setup file only `lym slack setup` writes, so no `lym config` write repoints a live relay.
  `login`, `setup`, and `start` (outside an orch's pane), `away`, `back` and `stop` refuse from a strand's pane by creel's pane-ancestry check; the refusal guards a mistaken run, not a determined agent.
- The relay starts through a double fork so it sits in no pane's process tree at its first creel read, and creel reads it as no strand.
  Its first act is a creel read as `<shortname>:SLACK`; a refused first read from a non-orch pane ends it, and any other failure retries with backoff.
- Inbound is at-least-once around a crash: a creel post comes before the cursor write.
  A failing post holds later messages for up to 5 minutes, then the relay answers in Slack that the message was not delivered and moves on.
  An error from `Hub.Paused` after an accepted post moves the cursor, because retrying would forward the message twice.
- Outbound keeps a record of the creel ids already posted or skipped under `.lym/slack/`, so a read released before creel's acknowledgement deadline reposts nothing.
  Slack refusing a message five times with `ok: false` replaces it with one line naming its id and sender.
  Mail stamped more than 24 hours before a read is acknowledged without a post, and one line counts the skipped mail.
- The relay's restart is capped by lifetime: a relay that lives under 10 minutes counts one failure, and five in a row end in `gave-up`.
- The polling floor is 5 seconds, doubling to 60 on empty polls; a `429` waits out `Retry-After` and doubles that relay's floor for the rest of its run.

## Not included

The operator's later request on the board entry is not in this diff: presence switching by itself when the operator writes on Slack or types in an orch's pane, and the presence toggle in reed.
Presence changes only through `lym slack away|back`, the Slack texts and `lym slack stop`.

## To check by hand

None of this was run against a real Slack workspace; the tests use a fake Slack API.

- Run `lym slack login` and `lym slack setup` against a real workspace and channel, then `lym slack away`, and confirm a message you send reaches the prime orch and the orch's reply reaches the channel.
- Confirm the app is installed with the scopes `channels:history` and `channels:read` (or `groups:history` and `groups:read`), `users:read` and `chat:write`; the workspace may require admin approval first.
- Review the findings the reviews left open:
  - `relay-restart-cap-unreachable` in the relay's restart logic (`internal/slackengine/start.go`): the lifetime-based cap above answers it, but it was open at the plan's last review.
  - `setpresence-beams-error-outcome-unassigned`: what a presence change does when listing the hub's beams fails.
  - `stale-summary-lost-on-failed-read` in the outbound stale-mail line (`internal/slackengine/outbound.go`): the line naming skipped mail may be lost when its read fails.
