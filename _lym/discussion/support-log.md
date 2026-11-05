# Support log — reed-slack-relay

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Board: the entry asks for Slack as a creel endpoint, a presence state and reminders; open points are one channel or many, and token and user authentication.
  `pr-review-wait-silent` waits on this task for its reminder; `creel-cross-hub-mail` and `dormant-orch` touch creel and the orch without conflict.
- Code (explore subagents, then own reads): creel is per hub with no machine-wide registry; programs post through the Go client only; nothing models the operator as a sender; the orch reports to the operator only in its pane.
  Config has a machine layer; `lym config env` exports into every pane; githubclient keeps a machine-global credential file.
  `go.mod` has no Slack or WebSocket library.
  `loomshed.OperatorWaitRows` names Beam-Await, PR-Review and Sampler-Session.
- Slack facts (web research subagent): Socket Mode needs an app-level token; internal apps keep Tier 3 history limits; workspaces may require app approval.

### Turn 2 — design picks

- Channel per hub, app per machine (D1); token in a credential file, ids in machine config (D2; round 3 F1 moved the ids to a per-hub setup file); a per-hub relay process (D3); polling inbound (D4); outbound through the `SLACK` program mailbox (D5); presence file and notices (D6); orch role rules (D7); dobby reminders (D8); the CLI module (D9).

### Turn 3 — lens review

Every lens returned a report.

- walkthrough 1 / bound-breaker 4 (cursor never passes unforwarded messages): fixed, D4 moves the cursor past every message once handled.
- walkthrough 2 / interactions LOW / existing-paths LOW (relay identity depends on its starter's pane): fixed, D3 starts it through a double fork so it is outside every pane tree at its first read (review round 1 F1 corrected the earlier new-session claim).
- interactions 1 / existing-paths LOW / bound-breaker 5 (`SLACK` not a creel receiver before the relay's first read): fixed, D3 reads first, D6 `away` waits for that read, D8 logs a refused post.
- interactions 2 (presence notice reads as operator words): fixed, `[slack presence]` prefix in D6 and D7.
- interactions 3 (operator answer to a beam orch lands at the prime): fixed, D5 adds the `@<slug>` answer hint.
- interactions 4 / walkthrough 4 / bound-breaker 3 (setup needs read scopes): fixed, D2 lists `channels:read`/`groups:read` and `users:read`.
- walkthrough 3 (ack covers the whole read): fixed, D5 acknowledges only a fully posted read and releases otherwise.
- walkthrough 5 (dead relay while away cannot be restarted from the phone): fixed, D3 has the prime dobby's ticks start it while presence is `slack`.
- walkthrough 6 / bound-breaker 6 / existing-paths (wait start field): fixed, D8 keeps the wait's start in dobby's state record (review round 1 F7 replaced the history entry's `At`).
- existing-paths MEDIUM (no stop path or role for the relay): fixed, D3 gives it a role and `lym slack stop`; D9 lists the verb.
- existing-paths MEDIUM (pause or hold silently delays Slack mail): fixed for a pause, D4 answers in Slack and D3's status shows it; a hold stated as a D4 bound.
- existing-paths LOW (4–8 KiB body only reminded on a pane-mail orch): fixed, D4 caps at 4 KiB.
- existing-paths LOW (two machines on one channel): fixed, D2 bound and help text.
- bound-breaker 1 (authority bound overstated): fixed, D7 bound restated as creel's program trust; the Constraints line removed.
- bound-breaker 2 (any strand can post to Slack): fixed, D5 posts only orch and program mail.
- bound-breaker table LOW (presence change seen by the relay; re-exec lock): fixed, D4 reads presence at each poll, D3 takes the lock again after re-exec.
- bound-breaker table LOW / leanness 2 (Socket Mode spreading claim unsourced): fixed, removed from the record; kept here as the writer's understanding.
- board-scope 1 (`pattern-distill` folds cited IDs): fixed, Constraints line on reading IDs under clusters and placing the new entry.
- board-scope 2 (`pr-review-wait-silent`): no change needed, Open risks line kept.
- leanness 1 (auto-mode arguments): fixed, cut to the picks; arguments live under Rejected alternatives.
- leanness 3 (Notes duplicate relevant.md): fixed, section removed.
- leanness 4 (PATTERN restatements in Constraints): fixed, cut.
- leanness 5 (D9 and D3 rationale): fixed, D9 rationale dropped, D3 kept to one clause.
- leanness 6 (admin approval twice): fixed, kept in Open risks with its source.
- leanness 7 (D7 attach command wording): fixed.
- leanness 8 (Goal restates D6): fixed, the third sentence shortened.

## Rejected alternatives

- One channel for every hub, each message naming its hub: needs one machine-wide relay and a hub registry that lym lacks; replies would need a hub prefix the operator types on the phone.
- Socket Mode: needs an app-level token and a WebSocket dependency. The writer understands Slack to send each event to one of an app's open connections, so per-hub relays sharing one app would each miss events; not verified against Slack's docs, and not needed once polling was picked.
- Hosting the relay in the creel daemon: creel's engine decides delivery and imports no transport; a network client inside it breaks that layering.
- Token in `lym config env`: exported into every agent pane.
- A Slack post verb for agents (`lym slack post`): a second messaging path beside creel, against `PATTERN-agent-messaging`; replies would not reuse `--reply`.
- Go mirroring the orch's pane to Slack: the pane carries tool output and noise; what is meant for the operator is the orch's judgment.
- Reminders from the relay: it would need every worktree's run status, which dobby already reads.
- Reminding Beam-Await: it waits on members too, so most of its waits are not the operator's.
- A slack-go dependency: a few Web API methods over `net/http` need no library.
- Routing prefixes other than `@<slug>`: `/` opens Slack slash commands, `#` links channels and `>` quotes.
- Posting every strand's mail to Slack: any agent could then spam the operator's phone or push repository text out.

## Review rounds

### Round 3

- F1 (a `lym config slack --layer machine` write from the prime orch's pane repoints channel and operator past setup's checks): fixed differently from the proposal's `slack.json`, since that file is one per machine and the channel is per hub; D2 moves both ids to a per-hub setup file under `.lym/slack/` that only `lym slack setup` writes, and the `slack` config module keeps only `remind_after_min`.
- F2 (pane-ancestry check silent on an unknown pane process): fixed, D6 defines the check's refusal for that case as creel does, and D2 and D3 refer to it.
- F3 (`lym slack relay` by hand holds the lock): fixed, D3 ends the relay when creel refuses its first read as coming from a non-orch strand's pane, and retries any other refused first read holding the lock; the first read comes before any poll; D9 states its `internal` audience.
- Self-check: the unknown-pane refusal names only waiting, and covers unreadable strands; D6 bounds the verbs' refusal while a pane process stays unknown; D2's rationale and bound restated.

## Question ledger

- Q1 One channel or one per hub? — auto-pick: one per hub (D1).
- Q2 Where is the token kept? — auto-pick: machine credential file beside githubclient's cache (D2).
- Q3 How is the operator's Slack user authenticated? — auto-pick: configured user id, checked on every message (D2, D4).
- Q4 Socket Mode or polling? — auto-pick: polling (D4).
- Q5 Which process hosts the relay, and who restarts it? — auto-pick: its own per-hub process, started by the prime bring-up, `away`, `start` and the prime dobby's passes while away (D3).
- Q6 How does a message reach a beam's orch? — auto-pick: `@<slug>` prefix, with an answer hint on beam orch posts (D4, D5).
- Q7 How do agents post to Slack? — auto-pick: creel mail to `<shortname>:SLACK`, orchs and programs only (D5).
- Q8 How does a cycled orch know the presence? — auto-pick: notice on change plus a line in the resume pointer (D6).
- Q9 Who sends reminders, for which rows, how often? — auto-pick: dobby, PR-Review and Sampler-Session, every 30 minutes by default (D8).
