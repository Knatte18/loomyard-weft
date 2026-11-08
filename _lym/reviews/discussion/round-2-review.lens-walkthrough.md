```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's own purpose.

**Decisions walked**
- D1: hubs `ly` and `mh` on `neve`, with `ly`'s relay holding the lock. An operator message in `mh-neve-prime` becomes an event, the holder reads both maps, writes the hint `C_mh` into mh's inbox and acks, and mh reads its history. **Purpose met.** The hint-write failure, lock hand-over and re-exec paths also hold.
- D2: a hint `C1` arrives while the relay drains its inbox. The record never orders "remove the hint" against "read history", and one order loses the hint (F1). A thread reply to an operator's top-level message is also walked (F4). **Purpose met** only with remove-before-read.
- D3:
  - **Fresh-machine login of `xoxb` then `xapp` then `xoxp`:** purpose met.
  - **Bot token logged in after an `xapp` is stored:** the match needs the app-level token's app id, which the record neither stores nor says how to obtain (F5).
  - **Relay re-exec'd with only a bot token:** it sends one notice and exits, and dobby then gets `OutcomeNotConfigured`. Purpose met.
- D4:
  - **Names:** `ly` + `neve` + `slack-v2` gives `ly-neve-slack-v2`.
  - **Two hubs with different shortnames on one machine:** they cannot produce the same name, because the shortname grammar `[a-z][a-z0-9]{1,5}` has no `-` (`internal/agentname/agentname.go:88-93`).
  - **Purpose met.**
- D5:
  - **Prime ensured at start:** create, then write the entry with an empty cursor, then invite the operator. Purpose met when every call succeeds.
  - **Operator invite fails after the entry is written:** it is never retried (F3).
  - **`name_taken` on an archived channel this bot created, with no map entry:** the reopen's cursor is undefined (F2).
  - **Live channel this bot created but is no longer a member of:** refused with no stated way forward (F6).
- D6: beam `x` lands, the next wake archives its channel, and its later mail posts to the prime's channel. A new beam `x` then reopens the channel with its cursors kept, and `lym slack stop` archives after the relay ends. **Purpose met** whenever an entry exists.
- D7: "hi" in `ly-neve-slack-v2` is forwarded to `ly:slack-v2` orch, and `away` in a beam channel is answered there. **Purpose met.**
- D8: mail from `ly:slack-v2:LOOM` gets its slug from `programSlug`, then the map entry or an ensure, then the post with `username`. The `is_archived`, `not_in_channel` and `channel_not_found` re-posts were walked. **Purpose met.**
- D9: `SetPresence` notifies the prime and every beam. **Purpose met.**
- D10: every call the decisions make maps to a manifest scope: create, archive and invite to `channels:manage`; list to `channels:read`; `bots.info` and `users.info` to `users:read`; unarchive and the user invite to the user scope `channels:write`; `apps.connections.open` to `connections:write`. **Purpose met.**
- D11: the `hello`, `events_api` and `disconnect` envelopes pass through the seam, and the ack is by `envelope_id`. **Purpose met.**

**Findings**

F1. MEDIUM, design, D2 (inbox drain), `decision-record.md:113`, with `:106` and `:125-130`.
- **What breaks:** the record never says whether the drain removes a hint before or after the Slack read it triggers. If the read comes first, this happens:
  1. The relay reads history for hint `C1`.
  2. The operator's next message M2 lands.
  3. The holder overwrites `C1`.
  4. The relay removes `C1`.
- **End state:** M2 waits until the next resync or start, which can be hours away (Constraints, `:381`). The record's coverage claim does not hold for the operator's last message.
- **The rationale is also wrong:** "The gaps push leaves are a relay not running, the lock hand-over and a reconnect" (`:128`) leaves this gap out.
- **Fix:** have the decision require that each hint is removed before its read.

F2. MEDIUM, design, D5 step 2 together with D6, `decision-record.md:226-230` and `:260-263`.
- **Case:** the map holds no entry for the channel, as after D3's way forward of moving an unreadable map aside, or when the ensuring hub never held that channel. `name_taken` then finds an archived channel this bot created, and D5 reopens it "(D6)".
- **The gap:** D6's reopen keeps "the cursors … where the entry left them", but no entry exists. D5 sets a cursor only for create (empty) and adopt (newest).
- **End state:** if the implementer follows the create path's empty cursor, the first read forwards every operator message in the old channel's history to the orch as new mail. That breaks D2's "no reply the channel already handled is forwarded again" (`:93`).
- **Fix:** state that this reopen sets the cursor and thread floor at the channel's newest message, as adopt does.

F3. MEDIUM, design, D5 step 3 and the retry rule, `decision-record.md:225`, `:236` and `:238-239`.
- **Case:** `conversations.create` succeeds and the entry is written. The operator invite then fails on a transport error or a rate limit.
- **The gap:** D5's retry covers only "a channel the map does not hold" and a refused channel. This channel is in the map, so no later ensure runs and the invite is never retried.
- **End state:** the operator is never invited, which misses "invites the operator" (Scope `:17`) and Rationale `:244`.
- **Fix:** mark the entry as owing an invite and retry it at each ensure or wake until it stands.

F4. LOW, design, D2 thread read, `decision-record.md:101`.
- **Case:** a thread read with `conversations.replies` for a thread whose parent is the operator's own top-level message.
- **The gap:** the record does not say that the thread read skips the parent. Slack's `conversations.replies` answer may include the parent message as its first item; I did not verify whether it does so when `oldest` excludes it.
- **End state:** if it does, the parent would be forwarded a second time.
- **Fix:** have the decision skip a message whose `ts` equals its `thread_ts` in a thread read, or cite Slack's behaviour as evidence.

F5. LOW, design, D3 login match, `decision-record.md:147-153`.
- **Case:** `xapp` is stored first, then a bot token is logged in.
- **The gap:** the step "a bot token that does not match the stored tokens replaces…" needs the stored app-level token's app id. The record gets that id only from a live connection's `hello` (`:143`), and it neither stores the id in `slack.json` nor says the bot-token login opens a connection with the stored `xapp`.
- **End state:** the step's input is undefined.
- **Fix:** say which of the two the login does.

F6. LOW, design, D5 step 2 refusal, `decision-record.md:228-234`.
- **Case:** `name_taken` finds a live channel this bot created and that no map holds, but the operator has removed the bot from it.
- **The gap:** it falls under "Any other is refused". The notice names a way forward only when another hub's map holds the channel or another bot created it, so the operator gets no way forward.
- **End state:** this breaks the decision's own notice rule and `PATTERN-refusal-way-forward`.
- **Fix:** invite the bot back with the user token, under D3's creator bound, then adopt.

**Tool calls**
1. `grep` for `ValidateShortname` in `internal/agentname`: my context had no shortname grammar, and I needed it to test whether two hubs on one machine can collide on a D4 name.
