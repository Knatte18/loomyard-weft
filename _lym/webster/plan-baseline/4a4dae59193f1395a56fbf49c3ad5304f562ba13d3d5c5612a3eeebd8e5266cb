# Card 14 — orch-role-slack

**Extends:**
- `contracts/orchrole/orch-role.md`

**Intent:** Give the orch role its Slack rules (D7), in a `## Slack` section after `## Messaging`, on top of creel's landed sender and trust rules:

- Mail from `<shortname>:SLACK` whose body opens `[slack]` carries the operator's own words, with the same authority as a line typed in the orch's pane; answer it with `lym creel post --reply <id> "<text>"`, which reaches Slack.
- Mail opening `[slack presence]` tells the new presence and is not answered.
- While presence is `slack`, every message meant for the operator — a design question, a go-ahead the orch needs, a status report, a relayed owner halt and an answer to the operator — is also posted to `<shortname>:SLACK` with `lym creel post`, leaving out the attach command, which needs a terminal.
- While presence is `terminal`, the orch posts to `<shortname>:SLACK` only replies to mail that came from it.
- A beam orch's post to `<shortname>:SLACK` reaches the channel with an `@<slug>` answer hint, and the operator's answer comes back to that beam's orch.
- The presence is learned from `[slack presence]` mail and from the resume pointer's presence line; `lym slack status` prints it.

The role carries no new marker, so no Go fill or test changes; the wording is the code review's to check.

**ImpactSummary:** One added section of the embedded orch role text; no marker or Go reader changes.
