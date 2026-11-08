# Slack v2: one bot, one channel per orch, push in place of polling

`slack-v2` · Note · sketch, slack, creel, orch

Decision record: [Sketch](decision-slack-v2.md)

## Changed after the Discussion (operator, 2026-11-08)

These override the decision record where they differ (D3, D6):

- No operator user token (`xoxp-`): each machine stores two tokens, the bot token (`xoxb-`) and the app-level token (`xapp-`), and `lym slack login` asks for both in one run.
- A channel is archived only when its orch retires, that is when its beam is gone and its project done; `lym slack stop` archives nothing, since orchs are long-lived.
- Before archiving, the bot renames the channel with a date suffix, such as `ly-neve-<beam>-2026-11-08`, so its name is free again and its history stays searchable in the archived channel.
- An archived channel is never reopened: an orch that needs Slack under a freed name gets a new channel.

## Seen (operator and orch, 2026-11-07, first live Slack test)

- The relay polls `conversations.history`, from every 5 s up to every 60 s when the channel is quiet, so the operator's message reached the orch about 10 s after it was written; the operator finds that too slow.
- A hub has one channel; a beam orch's post reaches it with an `@<slug>` answer hint, and the operator must write `@<slug>` to reach that orch, which grows messy with many beams.
- The operator wants one Slack bot, and to reach each orch by writing in that orch's own channel.
- The first live post repeated four times (`slack-post-decode-repost`, a task of its own, fixes that).

## Wanted (operator, 2026-11-07)

- One Slack workspace for every use of lym: every hub, every repository and every machine the operator works on, Models included, since switching workspaces is a chore (operator, 2026-11-07).
- The workspace is the operator's alone; anyone else using lym makes a workspace of their own (operator, 2026-11-07).
- One Slack app and bot per machine, such as `lym-neve` and `lym-osl`, shared by every hub on that machine, never one per agent or hub; each machine stores its own app's tokens through `lym slack login` (operator, 2026-11-07).
- Channel names tell hub and orch apart across that one workspace, and two hubs or two machines never relay one channel.
- One channel per orch: the prime's orch and each beam's orch.
  What the operator writes in a channel reaches that channel's orch, and that orch's posts go to its channel.
  Every orch posts to the one address `<hub>:SLACK` as today; lym looks up the channel of the mail's sending orch, so no orch knows its channel (operator, 2026-11-07).
  The operator never removes a channel by hand: lym archives a channel itself when its orch retires, as the section above says.
  The bot makes each channel itself when it is first needed and invites the operator, so the operator makes no channel by hand (operator, 2026-11-07).
- Messages reach the orch within about a second: Socket Mode or another push route in place of polling, with the extra token it needs stored as the bot token is.
- Each orch's posts show its name, and an icon if cheap, through `chat:write.customize`.
- The app manifest lym documents lists every scope this needs.
