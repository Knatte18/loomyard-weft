# Slack v2: one bot, one channel per orch, push in place of polling

`slack-v2` · Task · feature, slack, creel, orch · draft

## Seen (operator and orch, 2026-11-07, first live Slack test)

- The relay polls `conversations.history`, from every 5 s up to every 60 s when the channel is quiet, so the operator's message reached the orch about 10 s after it was written; the operator finds that too slow.
- A hub has one channel; a beam orch's post reaches it with an `@<slug>` answer hint, and the operator must write `@<slug>` to reach that orch, which grows messy with many beams.
- The operator wants one Slack bot, and to reach each orch by writing in that orch's own channel.
- The first live post repeated four times (`slack-post-decode-repost`, a task of its own, fixes that).

## Wanted (operator, 2026-11-07)

- One Slack workspace for every use of lym: every hub, every repository and every machine the operator works on, Models included, since switching workspaces is a chore (operator, 2026-11-07).
- One Slack app and one bot for that workspace, shared by every hub, never one per agent or hub; each machine stores the same bot token through `lym slack login`.
- Channel names tell hub and orch apart across that one workspace, and two hubs or two machines never relay one channel.
- One channel per orch: the prime's orch and each beam's orch.
  What the operator writes in a channel reaches that channel's orch, and that orch's posts go to its channel.
  The bot makes each channel itself when it is first needed, invites the operator, and removes it when the relay stops, so the operator makes no channel by hand (operator, 2026-11-07).
  A bot token can archive a channel but not delete it; an archived channel keeps its name, so a channel made again under the same name needs the archived one renamed first, or unarchived where the token allows it.
- Messages reach the orch within about a second: Socket Mode or another push route in place of polling, with the extra token it needs stored as the bot token is.
- Each orch's posts show its name, and an icon if cheap, through `chat:write.customize`.
- The app manifest lym documents lists every scope this needs.

## Open for the Discussion

- The channel name scheme: hub, orch (prime or beam slug), and whether the machine is part of it, which it must be when one hub runs on two machines at once, each with its own prime orch.
- Whether the prime's channel is removed at relay stop like a beam's, or stays, so its history is at hand.
- Whether the prime's orch uses the bot's direct message instead of a channel.
- How presence works with many channels: one presence for the hub, or one per orch.
- What happens to a beam channel's history at teardown, and to mail queued for a channel that no longer exists.
- Whether `slack-presence-auto` (the presence switch by where the operator writes, and the reed toggle) joins this design.
- What else a revision should settle: other machines' hubs in one workspace, a second operator, files and images, threads.
