```yaml
lens: board-scope
outcome: reported
```

Task: the board-scope lens on the slack-v2 decision record and support log.

**Findings**

1. `LOW` · D8 (Scope "Out", Auto-mode assumptions) · **The record drops the icon without weighing its cost.**
   - Class: design.
   - The entry wants "Each orch's posts show its name, and an icon if cheap, through `chat:write.customize`."
   - The record puts "a custom post icon" out of scope. The support log's only reason is "no need named beyond 'if cheap'".
   - Slack's docs say `icon_emoji` is one more field on the same `chat.postMessage` call D8 already makes. So the cost condition the entry sets is met, and the record never argues it isn't.
   - Fix: either add a fixed `icon_emoji` per sender kind to D8, or state why it is not cheap.
2. `LOW` · D3, Open risks · **The two runs can disagree on what `login`'s prompt asks for.**
   - Class: design.
   - `slack-login-prompt` (T3, running Darn-Write) wants `login` to print "a prompt naming what to paste". Its decision record exists but is still the bare template: its Goal quotes the entry, and every other section is empty.
   - D3 makes the same verb take three token kinds, filed by prefix. T3 will most likely prompt for "the bot token", which would be wrong once D3 lands.
   - The record's Open risk treats this as a rebase-only overlap. It is a small behaviour overlap.
   - Fix: D3 states that the prompt names the three kinds it takes, so whichever run lands second knows what to reconcile.

**Goal points of the entry**

- One workspace for every hub, repository and machine: met (D4 machine segment, D10 one app per machine).
- The workspace is the operator's alone: met, since nothing in the record is multi-user.
- One app and bot per machine, with tokens through `lym slack login`: met (D3, D10).
- Channel names tell hub and orch apart, and two hubs or machines never relay one channel: met (D4, D1).
- One channel per orch, the operator's text reaches that orch, and every orch posts to `<hub>:SLACK` without knowing its channel: met (D5, D7, D8).
- lym archives beam channels itself, so the operator removes none by hand: met (D6).
- The bot makes each channel when first needed, invites the operator and removes it at relay stop: met (D5, D6). The prime's channel stays, which the entry lists as an open question and the record settles as an auto-pick.
- Reopen an archived channel with its history, and check whether a bot token can unarchive: met. The research shows bots cannot, so the user token reopens (D3, D6).
- About a second of latency through Socket Mode, with the extra token stored like the bot token: met (D1, D3).
- Each post shows the orch's name, with an icon if cheap: name met, icon not (finding 1).
- The manifest lists every scope: met (D10).

**Open questions of the entry**

- Socket Mode with several hubs on one machine: settled (D1).
- The channel name scheme: settled (D4).
- Whether the prime's channel is removed at stop: settled (D6).
- Whether the prime uses the bot's direct message: settled (auto-pick).
- Presence for many channels: settled (D9).
- Beam channel history and queued mail at teardown: settled (D6, D8).
- Whether `slack-presence-auto` joins: settled, it does not (D9).
- Files, images and threads: settled (D7).

**Entries, peers and commits checked**

- `slack-presence-auto` (F2, note): no conflict. The record keeps it separate, and the note's switch on an operator Slack message fits D7 and D9.
- `slack-login-prompt` (T3, peer): small overlap (finding 2).
- `go-test-cache-disk` (T2, peer): no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**: no `Decision for slack-v2:` note exists on the board, and the record has no `Added after Discussion` entries. I could reach everything I looked for.

**Tool calls**

- I read `/home/knatte/Code/loomyard-LYMHUB/slack-login-prompt/_lym/discussion/decision-record.md`. My context held only that peer's board entry, not its decision record. The board list, task entry and peers output were already in my context, so I made no other calls.
