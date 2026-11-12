# Support log — slack-machine-bridge

## Interview

Autonomous run: no operator answered, and the step has no advisors.
Every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — exploration

- Board: the entry's Idea and its section changed by `slack-per-orch-activation` were read; no `Decision for slack-machine-bridge:` note exists, and `lym loom peers` lists two peers, neither touching Slack or creel.
- Three code subagents mapped the relay (`internal/slackengine`, one relay per hub, one Socket Mode holder per machine), creel's cross-hub path and trust stamp, the machine name (`internal/machinename`) and the per-orch button.
- Facts settled from code: `agentname.FormatMachineAddress` already writes `<machine>::<name>` for display and no parser reads it; inbound today forwards only the operator's messages with no `bot_id`; bots.info and conversations.invite are already called.
- A research subagent read docs.slack.dev: scopes for create, invite, setPurpose, users.conversations, users.info and bots.info are documented;
  bot-to-bot message events, inviting another app's bot and message metadata's scope are not settled by the docs.

### Turn 2 — design questions, each auto-picked

1. Who may post across machines? Recommended: orch to orch only. Picked (D2).
2. How does creel hand a post to Slack? Recommended: journal as `outbound`, the relay takes and settles it through program-only Go-client verbs. Alternatives: mail to `<shortname>:SLACK` carrying an envelope; creel calling Slack. Picked the recommendation (D3).
3. How does the peer's bot reach the channel? Recommended: the creator invites it, with a computed name for races and a users.conversations resync. Alternative: the board's join-itself idea. Picked the recommendation (D4).
4. Where does the envelope ride? Recommended: a first text line. Alternative: message metadata. Picked the recommendation (D5).
5. How is a peer approved with no setup beyond Slack? Recommended: operator commands `approve` and `revoke` in Slack, pinning the app id. Alternatives: a `lym slack` verb; a config key. Picked the recommendation (D6).
6. What happens to mail from an unapproved app? Recommended: hold the cursor and notify once. Alternative: drop with a notice. Picked the recommendation (D8).
7. Does a bridge have its own switch? Recommended: no, it follows the orch's button at both ends. Picked (D10).
8. Does trust also need `trusted_hubs`? Recommended: no, machine approval and an orch sender alone. Picked (D9).

### Turn 3 — lens review

Six forks returned reports; none was missing. Duplicates across lenses are merged on one line.

- interactions 1 + bound-breaker 2 (BLOCKING): a forged envelope from an unpinned app held the cursor for good — fixed: a mismatched app is skipped with a notice once a pin exists; hold only while no pin exists (D8).
- walkthrough 1 (BLOCKING): a held cursor blocked the `approve` typed after it — fixed: operator messages are handled past a held peer message (D8).
- bound-breaker 1 + walkthrough 3 (BLOCKING/MEDIUM): dedupe and reply-to keyed on machine and id lost mail of two sending hubs — fixed: keyed on the sender's full address with its shortname (D8).
- interactions 2 + walkthrough 4 (MEDIUM/LOW): a lost settle or two hour-owners could return delivered mail — fixed: creel owns the hour, the relay reports hold reasons and re-settles from its outbox at start (D3, D7).
- existing-paths 1 (MEDIUM): creel's per-hub workers would pick up machine-outbound entries — fixed: the scheduler skips them, the take verb is their only reader (D3).
- existing-paths 7 (LOW): the Slack timestamp does not fit the integer remote id — fixed: delivered carries it in a field of its own (D3).
- bound-breaker 4 (MEDIUM): "no agent pane can make that claim" was false — fixed: bound names the orch pane and processes in no pane (D3).
- interactions 3 + existing-paths 5 (MEDIUM/LOW): the peer kept an archived channel id — fixed: an archived or missing channel is dropped from the map and found again (D4).
- interactions 4 + walkthrough 2 + bound-breaker 10 (MEDIUM/LOW): resync read the wrong end, or no envelope, or another hub's channel — fixed: the newest message with an envelope, the end naming this machine and this hub's shortname (D8).
- existing-paths 3 (MEDIUM): the holder sees no message text — fixed: an unmapped-channel event becomes a resync hint to every running relay (D8).
- bound-breaker 5 (MEDIUM): any member could grow the bridge map — fixed: a channel is admitted only when this bot or an approved peer's bot made it (D8).
- existing-paths 2 (MEDIUM): `retryOrGiveUp` would drop a held message after five minutes — fixed: bridge holds are per channel and outside it (D8).
- existing-paths 4 (MEDIUM): an older binary would misread machine inbound as other-hub mail — fixed: a new journal event, carried with `of`, read as an id only (D8).
- interactions 5 + existing-paths 6 (MEDIUM/LOW): pending mail when the button goes off was unstated — fixed: held with `button-off`, returned at the hour with that reason (D7, D3).
- bound-breaker 3 (MEDIUM): single-candidate approval trusts an early impostor — fixed: the answer names the app id, `status` shows the own app id, bound stated (D6).
- interactions 6 + walkthrough 5 (LOW): the untrusted stamp was unreachable — fixed: the parser does not judge the role; orch-only is enforced at post (D1, D2, D9).
- interactions 7 (LOW): re-reading a held message could keep the poll at its base — fixed: only newly delivered messages reset it (D10).
- interactions 8 + bound-breaker 9 (LOW): users.list unevidenced and bots.info misattributed to the relay — fixed: users.list's page read (users:read); bots.info attributed to `lym slack login` (D11).
- interactions 9 + bound-breaker 8 (LOW): former machine names — fixed: own former names are refused at post and accepted as this machine on receipt; a renamed peer is approved by app id (D1, D6, D8).
- bound-breaker 6 (LOW): `name_taken` lists the workspace — fixed: stated as one listing per pair (D4).
- bound-breaker 7 (LOW): sanitized names are not injective — fixed: bound stated, with the creator and envelope checks as its guard (D4).
- leanness (MEDIUM): rejected alternatives argued in D4 and D5 — fixed: cut, kept in Rejected alternatives.
- leanness (LOW): Open risks held settled facts — fixed: the docs gaps moved into D4 and D10, the rename into D6; the section reads None.
- leanness (LOW): Auto-mode assumptions restated decisions — fixed: shortened to references.
- leanness (LOW): Notes duplicated the daemon-wakeups constraint — fixed: dropped from Notes.
- leanness (LOW): generic PATTERN restatements in Constraints — fixed: refusal, test and config-layers bullets cut.
- leanness (LOW): garbled D3 criterion — fixed: rewritten as checkable behaviour.
- leanness (LOW): Goal restated mechanisms — fixed: trimmed to the purpose.
- leanness (LOW): Interview Turn 2 and the ledger overlap — rejected: the spec asks for both; the turn carries the alternatives and the ledger the rationale.
- board-scope 1 (LOW): D4 deviates from the board's join-itself sketch — rejected: a design choice with its reason in Rejected alternatives; no board contradiction.

## Rejected alternatives

- Mail to `<shortname>:SLACK` carrying the receiver in its body: it bypasses creel's outbound map, its hour and its return notice, and needs a second envelope.
- Creel calling Slack itself: breaks `PATTERN-agent-addressing`'s Slack boundary.
- The peer's bot joining the bridge by itself, as the board entry first sketched: the peer would have to list every public channel of the workspace to find it.
- Slack message metadata for the envelope: the docs name two different read scopes and an event subscription, and do not say it reaches a Socket Mode app.
- A `lym slack approve` CLI verb or a config key for peers: the operator asked for no setup beyond Slack; a config key would also put a trust pin in a layer.
- Dropping mail from an unapproved app: the first message between two machines would always be lost.
- Any strand posting across machines: one channel per pair of orchs would carry unrelated strands' traffic.
- A per-bridge switch: the board leaves it open; the orch's button already says whether the operator wants that orch on Slack.
- Requiring the sender's repository in `trusted_hubs`: the board asks for machine approval alone, and the bridge would need a second list kept on both machines.
- Relying on Socket Mode events alone to read bridges: the docs do not say Slack sends one bot's messages to another bot.

## Review rounds

- Round 1 (BLOCKING, 16 findings): every finding fixed.
  An adopted or resync-added channel reads from its start (D4); the hour's bound past a long relay outage and a late settle (D3);
  a bridge-post record apart from the outbox (D7); bridge-channel commands handled by one machine (D6);
  resync trusts only pinned envelopes, uses a bridge resync hint of its own and notifies on an unpinned creator (D8);
  the missing acceptance criteria added.
  F4's not-delivered answer stays in bridge channels, given once by the first end's relay, since an orch channel forwards operator text.
- Round 2 (BLOCKING, 15 findings): every finding fixed.
  A per-channel operator cursor handles each operator message once past a held message, and threads are read (D4, D8);
  an adopted channel skips its existing operator messages and envelopes older than creel's retention (D4, D8);
  a hand-archived channel holds as `bridge-archived` with a rename way forward (D4); a held message creel returned is not posted (D7);
  machine-outbound mail is an event of its own, taken at each bridge poll, carrying the sending id as reply-to (D3, D8, D10);
  the refusal count lives in the bridge-post record and the peer app leaves the bridge map (D4, D7).
  The once-per-pair listing of Turn 3 is corrected to its true rate (D4).
- Round 3 (BLOCKING, 4 findings): every finding fixed.
  A scheduler timer returns machine-outbound mail at its hour without the relay, and take never returns a message past it (D3);
  report, settle delivered and settle returned have journal events of their own, report writing only on a changed reason (D3);
  the bridge-archived way forward is to unarchive, and to rename only a channel no bot of the pair made, since Slack refuses renaming an archived channel (D4);
  notice marks live in a bridge-notice record keyed by pair or channel, and an approval moves the delivery cursor past what it delivers (D4, D8).
  The fixer's self-check added the pair-keyed marks, the unarchive-only path, the change-of-reason wording and an idempotent late settle (D3, D4).

## Question ledger

- Q1 (resolved, auto-pick): orch to orch only — keeps the bridge to the task's purpose and one channel per orch pair.
- Q2 (resolved, auto-pick): creel journals, the relay takes and settles — reuses the cross-hub outbound model and keeps Slack calls in the relay.
- Q3 (resolved, auto-pick): invite plus computed name — no workspace listing; the race and a refused invite each have a path.
- Q4 (resolved, auto-pick): envelope in the text — rests only on documented `chat:write`.
- Q5 (resolved, auto-pick): in-Slack `approve`/`revoke` pinning the app id — no setup beyond Slack, and an app named `lym-<machine>` by someone else is not trusted.
- Q6 (resolved, auto-pick): hold unapproved mail at the cursor — no lost first message.
- Q7 (resolved, auto-pick): bridges follow the buttons — the board's hint.
- Q8 (resolved, auto-pick): machine approval alone grants trust — the board's wording.
- Q9 (resolved, auto-pick): an operator message in a bridge channel is not mail — the orch's own channel carries the operator's words.
- Q10 (resolved, auto-pick): two Slack behaviours the docs leave unsettled, a bot inviting another app's bot and bot-to-bot message events — the design carries the bridge without either (D4's held invite, D10's poll).
