```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of the record's decisions that share state, a command or an ordering. Four pairs broke, one of them only weakly; the rest held. I made no further tool calls.

**Pairs checked**
- D3 relay start (prime bring-up) × D6 `lym slack away` start: both are a start that finds a live relay and does nothing. Held.
- D3 re-exec on a replaced binary × D4 cursor: the existing bound, a resend after a crash, covers it. Held.
- D3 re-exec × D5 unacknowledged read: creel releases the mail when the connection drops. Held.
- D4 inbound filter on `bot_id` × D5 the relay's own posts: the relay's posts are skipped. Held.
- D4 unescaping × D5 escaping. Held.
- D4 resetting the poll on an outbound post or a presence change × D5 and D6: all of it runs in the same process. Held.
- D4 presence commands × D6 verbs: same state, both answered in Slack. Held.
- D4 checking the `@<slug>` beam list × D6 notifying every beam's orch: both need the same list of beams, and they agree. Held.
- D4 default routing to the prime orch × D7 beam orchs mirroring to Slack: **broken**, finding 3.
- D5 `--reply` × D7 the orch answering Slack mail: a reply to a program's message reaches that program's mailbox. Held.
- D5 receiver `<shortname>:SLACK` × D6, D7 and D8 posting to it: **broken**, finding 1.
- D6 presence notice × D7 the authority grant for `[slack]` mail: **weak**, finding 2.
- D6 "starts over" × D8 reminder timing: the first reminder comes at once when the wait already passed the interval. Held, and it matches the purpose.
- D6 resume-pointer line × D7 the terminal/slack rule. Held.
- D7 terminal rule × D8 reminders sent only while `slack`. Held.
- D2 checks in `lym slack setup` × the Review criterion's scope list: **broken**, finding 4.
- D2 operator id × D4 user filter. Held.
- D3 detached start × the creel sender bound in Constraints: held on the prime path. There is a risk if a non-orch pane runs `away` (LOW note at the end).

**Findings**

1. **MEDIUM** — D3, D5, D6 (also D8). Nothing makes the relay read its mailbox once before the first post to it.
   - Evidence: creel accepts as a receiver only "the name of a program that has read once" (`internal/creelengine/doc.go`, Receivers).
   - On `lym slack away`, D6 sets presence and sends the notice, and the relay may not have reached its first read yet. The orch's mirrored post, or a D8 reminder, to `<shortname>:SLACK` is then refused as an unknown receiver.
   - The same happens when a relay starts but fails before its first read, for example on a revoked token. Presence is `slack` and every post to SLACK is refused.
   - Fix:
     - The relay's first act is a read that makes its mailbox known.
     - `lym slack away` sets `slack` only after the relay is live and has read once; otherwise it refuses.
     - D8 and the D7 rule treat a refused post as an outage that `lym slack status` shows.

2. **LOW** — D6 × D7. The presence notice also comes from `<shortname>:SLACK`, and D6 does not give its body's prefix.
   - D7 grants operator authority to SLACK mail "whose body opens `[slack]`".
   - If the notice body also opens `[slack]`, it reads as the operator's words. If the orch replies to it, the reply goes to Slack.
   - Fix: give the notice a distinct prefix, such as `[slack presence]`, and say the orch does not reply to it.

3. **MEDIUM** — D4 × D7. Under D7 a beam orch mirrors its design questions to Slack, but D4 routes any reply without a prefix to the prime orch.
   - The operator answering a beam orch's question on the phone must type `@<slug>`.
   - Otherwise the answer lands at the prime orch, which has no context for it, and the beam orch keeps waiting.
   - D5's `[<sender>]` prefix shows who asked but does not route the answer.
   - Fix, either of:
     - D5 posts a beam orch's message with the `@<slug>` hint.
     - D7 tells the prime orch to forward mail that answers a beam orch.

4. **MEDIUM** — D2 setup × the D2 Review criterion and Constraints. D2's checks need scopes the record never lists.
   - D2's `lym slack setup` calls `conversations.info`, which needs `channels:read` or `groups:read`, and `users.info`, which needs `users:read`. It also checks the bot's channel membership.
   - The Constraints and the Review criterion list only `channels:history`/`groups:history` and `chat:write`.
   - An app installed with just those scopes makes setup fail, so the two parts of the record contradict each other.
   - Fix, either of:
     - Add the read scopes to the Constraints and the help-text criterion.
     - Drop those checks and verify with a history call alone.

**LOW note** — D3: the record does not say the relay is spawned outside the caller's pane process tree.
- Creel walks the process ancestry, and a program claim is accepted only from an orch pane or a process in no pane.
- A relay started by `lym slack away` from a non-orch pane would have its `SLACK` claim refused.
- The fix is to state that it is detached the way the creel daemon is.

**Acceptance criteria against decisions**
- The D4, D5, D8 and D9 criteria are consistent with their decisions.
- The D6 criterion "mail every orch … on a change" is affected by finding 1.
- The D2 Review criterion is affected by finding 4.

Further tool calls: none.
