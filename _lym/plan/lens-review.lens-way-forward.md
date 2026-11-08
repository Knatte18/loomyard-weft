```yaml
lens: way-forward
outcome: reported
```

Task: the way-forward lens review of the Slack push-channel plan in `_lym/plan/`. I checked whether each refusal, stop and recovery step names a next step that can succeed from the state it leaves.

**Findings**

1. **MEDIUM**, Card 11 (D6, D8): a beam with no open channel blocks every later message in the batch.
   - When the beams read fails, `channelFor` returns `errNoOpenChannel` and the whole read is released. `postMessages` returns at its first failure (`internal/slackengine/outbound.go:167-171`).
   - Prime-orch mail behind that beam message waits too, and the next read hits the same failure. While the beams read keeps failing, all outbound mail stalls until the 24-hour stale cutoff.
   - D6 says only "other beam mail waits for the next read". No planned row puts a prime message after the blocked beam message.
   - Fix: skip the message without marking it handled and go on with the rest of the batch.

2. **MEDIUM**, Card 11 (D8): the dropped-message line has nowhere to post when the ensure was refused.
   - Card 11 counts a refused ensure (`errChannelRefused`) toward `refusalCap`. Once the cap is reached, "the line about a dropped or refused message posts to that message's channel".
   - That channel does not exist, so the line naming `lym creel log --to &lt;shortname&gt;:SLACK` never posts. For a refused prime ensure there is no channel at all.
   - Fix: post the line to the prime's channel, or say plainly that none can post.

3. **MEDIUM**, Card 12 (D3): following `missingTokenExit`'s way forward leaves the relay down.
   - The notice names only `lym slack login` (or `lym slack setup --operator &lt;user-id&gt;`), and the relay then exits.
   - Login ends relays only when a stored token changes. By then this relay is already gone, so login starts nothing.
   - `EnsureRelayWhileAway` starts nothing while the presence is `terminal`. The relay comes back only at an orch bring-up or a `lym slack start` the notice never names.
   - Fix: add "then `lym slack start`" to the notice, and assert it in the reworked `HubWithNoTokenOrNoSetupIsRefusedAndTakesNoLock` row.

4. **MEDIUM**, Card 5 (D3): if the relay-ending step fails, re-running login cannot repeat it.
   - When the live-hub read fails after the tokens are written, the answer names the hubs record and its way forward. The tokens stay written.
   - After the record is repaired, re-running login with the same tokens finds nothing "stored and differs", so it ends no relay. Relays still holding the old tokens keep running.
   - Fix: the refusal should name `lym slack stop` and `lym slack start` in each hub as the next step. Alternatively, login could end the listed relays whenever the stored pair differs from what each relay last read.

5. **LOW**, Card 11 (D8): the refusal cap drops a message before the notice's fix can land.
   - A post refused with `is_archived` triggers an ensure. That ensure can meet `name_taken` from a channel the operator archived by hand, and the notice says to rename that channel in Slack.
   - Each refused read counts once, and a refused read waits only the floor before the next. So the message is dropped after five reads, about 25 seconds, before an operator can act.

6. **LOW**, Card 9 (D6): a retire whose rename stood but whose archive was refused is retried from the D4 name.
   - Card 9 records the new name only after the archive, so the entry keeps its D4 name while it stays open.
   - The retry renames again from the D4 name, which can yield `-2` names or a second dated rename.
   - If the slug is listed again meanwhile, `ensure` treats the open entry as held and posts into the dated, half-retired channel.
   - Fix: record the new name in the entry as soon as the rename stands.

7. **LOW**, Card 9 (D5): one refusal names no way forward.
   - A `name_taken` channel held by another orch of this hub "is refused with one notice naming the channel", with no next step. `PATTERN-refusal-way-forward` asks every new relay refusal to name one.

8. **LOW**, Card 5 and Card 8 (D1, D3): a failure to build a hub's geometry is reported as a broken hubs record.
   - `liveHubGeometries` fails as a whole when any hub's `hubgeom.SlackGeometry` fails, for example when `machinename.Name` cannot read the hostname, which card 4 makes a refusal.
   - The holder's notice then says to move the hubs record aside. That cannot fix a geometry failure, and the record is re-recorded unchanged.

9. **LOW**, Card 5 (D3 as the operator entry amends it): a partial-token refusal leads the operator into a second refusal.
   - `RequireConfigured` names only the missing kind and `lym slack login`. A machine holding only the bot token is told it lacks the app-level token.
   - Login then refuses a run with one token, naming both kinds. The refusal should say that login takes both tokens in one run.

10. **LOW**, Card 12 (D5): a transient failure of the prime-channel ensure at start is never retried.
    - `relayStart` remembers only a refused prime ensure for the outbound wakes.
    - A transport failure at start, such as no network at boot, leaves no prime channel until the orch's next outbound post. Until then the operator has no channel to write in.

11. **LOW**, Card 10: one Slack refusal notice is shared across all channels.
    - `readFailed`'s once-per-episode notice keys on the single `apiNoticed` flag, which any successful read clears (`internal/slackengine/inbound.go:239`).
    - With one channel always refused and another reading fine, the notice is mailed again at every resync.

12. **LOW**, Card 4 and Card 6: an undecodable channel map gets no way forward in `lym slack status`.
    - `ReadChannelMap`'s decode error names the file but no next step, and `status` returns it as is. Card 9 adds "move that file aside" only for the keeper's notice.

**Tool calls:** none; the plan, the decision record and the code were already in context.
