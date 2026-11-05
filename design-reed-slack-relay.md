# Talk to the orch through Slack, and let the orch know when the operator is on Slack only

`reed-slack-relay` · Task · feature, reed · running · Discussion-Burler-Fix

## Need (operator, 2026-10-03, confirmed 2026-11-05)

The work machine runs the employer's Claude account, which the operator cannot reach from the phone at all; Slack can be on the phone.
The need has grown since 2026-10-03: the orchs on the work machine can be followed and steered from the phone only through Slack.

## Idea

Slack is an endpoint of creel, the message layer every agent already uses:

- A message the operator writes in the hub's Slack channel reaches the orch as creel mail from the operator, carrying the operator's authority as a line typed in the orch's pane does.
- The orch's replies to that mail, and its reports to the operator, go back to the Slack channel.
- Reminders reach the operator the same way, such as "this PR-Review has waited 30 minutes for you", repeated while the wait lasts (see `pr-review-wait-silent`).

## Operator presence

The orch knows whether the operator is at the terminal or reachable only through Slack:

- A presence state per hub, `terminal` or `slack`, set by a `lym` verb and by a short command written in Slack itself, such as "away" and "back".
- While the state is `slack`, the orch writes every message meant for the operator to Slack as well as its pane, and the reminders run.
- While it is `terminal`, Slack gets nothing but what the operator asked for there.

## Open

- Whether one Slack channel serves every hub on the machine (loomyard, `cs`, `lyt`), each message naming its hub, or each hub has its own channel.
- How the relay authenticates the operator's Slack user and keeps the Slack token out of the repositories.
