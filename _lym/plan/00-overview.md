---
format: 5
approved: false
language: go
---

# Plan: Slack relay per hub

Build the per-hub Slack relay: a machine token file and a per-hub setup file only `lym slack login` and `lym slack setup` write, a `lym slack relay` process started through a double fork that forwards the configured operator's channel messages to the hub's orchs as `[slack]` creel mail from `<shortname>:SLACK` and posts orch and program mail for that program to the channel, a presence state with its notices and resume-pointer line, the `slack` config module with `remind_after_min`, the relay starts at the prime's bring-up, its dobby's ticks and a prime anchor change, the `lym slack` verbs, the orch role's Slack rules, and the two PATTERN changes folded into their live clusters.

## Card Index

1 — pattern-clusters — fold the double-fork exception and the Slack boundary into PATTERN-child-processes and PATTERN-agent-addressing
2 — slack-names — declare the Slack directory name, the relay's runproc role and githubclient's credential directory export
3 — creel-seams — export creel's pane walk and orch test, code the program-claim-in-strand-pane refusal, add ReadWait and HubStrands
4 — slackengine-files — create slackengine with its geometry, token, setup, presence and relay-state files, and hubgeom's teller
5 — slack-config — add the hub-wide slack config module holding remind_after_min
6 — slackengine-api — add the Slack Web API client and its failure kinds
7 — slackengine-inbound — add the inbound poll, routing, presence commands and notices
8 — slackengine-outbound — add the outbound read that posts SLACK mail to the channel
9 — slackengine-relay — add the relay's run loop, double-fork start, ensure, stop and waits
10 — slack-cli — add the lym slack module with its pane check and verbs, mounted in newRoot
11 — orch-relay-presence — start the relay at the prime's bring-up and carry the presence in the resume pointer
12 — dobby-relay-tick — ensure the relay on each prime dobby tick while presence is slack
13 — anchor-relay-restart — start the relay again when a prime anchor change ends
14 — orch-role-slack — add the orch role's Slack rules

## Shared Decisions

### Decision: PATTERN changes fold into the live clusters

- **Decision:** Card 1 fences overview lines for `PATTERN-child-processes` and `PATTERN-agent-addressing` only, with the texts the parent's two Added-after-Discussion decisions record. No card fences a line for `PATTERN-reap-children` or `PATTERN-slack-boundary`, which those decisions withdraw: no top-level entry for either is added or changed.
- **Rationale:** The decision record's Delivers line and Constraints fold both rules into the clusters since pattern-distill landed, and the parent recorded the fold and removed the two withdrawn bullets from the record's `## PATTERN changes`, which now holds only the two cluster changes this overview copies.
- **Applies to:** card 1, and every card that departs from the old texts.

### Decision: slackengine is told its seams; slackcli wires them

- **Decision:** `internal/slackengine` holds every Slack call, the files, the relay loop and its start, and imports no `*cli` package; it is told its `Geometry` by `hubgeom.SlackGeometry` and its creel mailbox, Slack client, hub readers and clock as seams. `internal/slackcli` wires the creel program client (`creelcli.NewProgramClient` with `slackengine.SlackProgram`), the pane check and the hub readers. `orchcli`, `beamcli` and `fabriccli` start the relay only through `slackengine.EnsureRelay` or `EnsureRelayWhileAway`, which never refuse for a caller's pane, so a start from inside a non-orch strand's pane still brings up a relay the double fork has moved out of that pane's tree.
- **Rationale:** `PATTERN-cli-cobra`'s engine-never-imports-cli rule, `PATTERN-cwd-resolution`'s told geometry, and dobbyengine's precedent of an engine spawning its own daemon and refusing to spawn under test.
- **Applies to:** cards 4 to 13.

### Decision: the relay's files and process

- **Decision:** The hub's `.lym/slack/` (`lymdirs.SlackDirName` under `fabricengine.HubScratchDir`) holds `setup.json`, `presence.json`, `cursor.json`, `outbox.json`, `relay.json`, the relay lock and the relay log; the token is `slack.json` in `githubclient.CredentialDir()`. The relay registers itself under `runproc.HubOwner` with `runproc.RoleSlackRelay`. Every relay argv carries `--hub-path <hub>`; the intermediate runs `--fork`, the relay `--after <intermediate pid>`.
- **Rationale:** D2 and D3; the creel daemon's `--hub-path` and dobby's self-registration are the precedents.
- **Applies to:** cards 2, 4, 9, 10.

### Decision: D8's reminders are outside this part

- **Decision:** This part ships the `slack` config module and its `remind_after_min` loader (Delivers, Exports); no card reads the value to send a reminder.
- **Rationale:** The record carries no D8 decision text, only the module and loader this part delivers.
- **Applies to:** card 5.

## PATTERN changes

- `PATTERN-child-processes` — changes: Starting or ending a process: log the spawn via `internal/logger`; its starter waits on it, bar a double fork's intermediate, which exits at once; a run's processes register in `internal/runproc` and end only through it.
- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; Slack only via `internal/slackengine`, its token only in its machine credential file; operator Slack text reaches agents only as creel mail from `<shortname>:SLACK`.
