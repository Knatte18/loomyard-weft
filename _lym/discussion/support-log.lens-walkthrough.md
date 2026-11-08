```yaml
lens: walkthrough
outcome: reported
```

Task: walk each decision's mechanism end to end through the walkthrough lens, then report the findings.

**Findings**

1. **BLOCKING** — D1 and D2: Slack's retry of an event that wasn't acknowledged is dropped by the cursor skip.
   - Case: in `ly-neve-slack-v2`, the inbox write for operator message ts=4 fails, so the envelope stays unacknowledged (D1).
   - Event ts=5 is then written and handled, and the cursor moves to 5.
   - Slack's retry of ts=4 arrives and is skipped as "not past its channel's cursor" (D2).
   - No reconnect happens, so no catch-up runs, and message 4 is lost.
   - The skip-by-cursor rule assumes events arrive in order. D1's retry path breaks that assumption. Missed events need a catch-up before the cursor moves, or a skip keyed on ts already handled, not on ts ≤ cursor.

2. **MEDIUM** — D2: "handled in write order" rests on a file-naming order the record never defines.
   - The inbox is a directory of files, so the relay reads them in name order.
   - Case: a retried envelope whose event_ts is older than a resync marker written just after a reconnect.
   - If files are named by event ts, the retried event sorts before the marker, and the stated guarantee ("a marker's catch-up runs before the events written after it") fails.
   - The record should name the ordering key, a holder-assigned sequence or write time.

3. **MEDIUM** — D4 and D5: two orchs can end up sharing one channel.
   - Case: hub `ly` has a beam with slug `prime` (named `ly-neve-beam-prime` by D4) and a beam with slug `beam-prime` (also `ly-neve-beam-prime`).
   - The second create hits `name_taken`. D5 step 2 finds a live channel the bot belongs to and adopts it.
   - The map then gives one channel id to two orchs, so D7's routing by channel is ambiguous.
   - Step 2 must refuse a channel the hub's own map already gives to another orch. D4's special case also needs a name the slug grammar cannot reach.

4. **MEDIUM** — D5: "presence becomes `slack`" has no trigger once the poll is gone.
   - Case: the operator runs `lym slack away` in a terminal. That CLI process writes `presence.json` and sends notices.
   - Today the relay notices the change through `inbound.presenceChanged` on each poll, and the poll is removed.
   - D1 wakes the relay only on its inbox. So the "ensure every live beam orch's channel" step never runs, or runs only at an outbound wake the record doesn't tie to presence.
   - The record should name the wake: an `internal/fswatch` event on `presence.json`, or a read at every outbound wake.

5. **MEDIUM** — D6 and D8: `lym slack stop` archives while the relay still runs, and D8 undoes it.
   - Case: `stop` archives `ly-neve-slack-v2`. Before the relay ends, the beam orch's mail arrives.
   - D8 gets `is_archived`, reopens the channel with the user token and posts.
   - The relay then ends with that beam channel open, which defeats "`lym slack stop` archives every open beam channel".
   - The order should be: end the relay, then archive.

6. **MEDIUM** — D10: the manifest drops a scope that setup still needs.
   - `setup --operator` still confirms the operator: the existing `UserInfo` in `internal/slackengine/api.go` calls `users.info`.
   - `users.info` needs the bot scope `users:read`, which D10's scope list omits, so setup would be refused on a fresh app.

7. **LOW** — D7: messages with files are skipped before the file rule can apply.
   - Slack has delivered an operator's file post with subtype `file_share`, and a thread reply sent to the channel with subtype `thread_broadcast`.
   - D7 skips "any other subtype ... as today", so neither reaches the file-naming or thread-forwarding rule.
   - The subtypes to admit should be named.

8. **LOW** — D6: the reopen's bot invite can fail on a bot that is already a member.
   - If archiving keeps the bot as a member, `conversations.invite` returns `already_in_channel`.
   - D5 treats that answer as done only for the operator's invite. D6 should say the same for the bot's invite.
   - The user scope D10 grants for that invite (`channels:write`) is also not confirmed against `conversations.invite`'s user-token scopes.

9. **LOW** — D2: resync markers pile up in hubs with no relay running.
   - The holder writes a marker into every live hub's inbox on every reconnect.
   - A hub whose relay is stopped collects one marker every few hours with nothing to clear them.
   - Markers could go only to hubs with a running relay, or be collapsed into one.

**Walkthrough list**

- D1 — event for hub B read by hub A's holder: purpose met, except retries (finding 1).
- D2 — reconnect followed by a retried old event: not met (findings 1 and 2).
- D3 — `xapp-` token pasted into `login`: met.
- D4 — slugs `prime` and `beam-prime` on hub `ly`, machine `neve`: not met (finding 3).
- D5 — first post from `ly:slack-v2:orch`: met. `lym slack away` from a terminal: not met (finding 4).
- D6 — beam torn down, then late mail from its orch: met. `stop` racing the relay's post: not met (finding 5).
- D7 — "hi" in a beam channel: met. Files and thread broadcasts: not met (finding 7).
- D8 — `ly:LOOM` to the prime's channel, a beam orch to its own channel: met.
- D9 — `away` written in a beam channel: met.
- D10 — fresh app, `setup --operator`: not met (finding 6).
- D11 — client against an `httptest` server: met; this is a dependency choice with no data flow to trace.

**Tool calls**

- Grepped `internal/slackengine/api.go` and dobby's files for the Slack methods called and who signs reminders. My context lacked the method list behind setup's operator check.
