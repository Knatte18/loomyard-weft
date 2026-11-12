```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on shared state, then checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D3 (creel owns the hour, re-settle at start) × D10 (relay runs only while some button is on): **broken**
- D3 hour × D7 hold reasons (`peer-unapproved`, `button-off`, D4 codes): held
- D4 refused-invite hold × D7 five-refusal return: **broken**
- D4 `name_taken` "listing once per pair" × D7 held `bridge-name-taken` retried: **broken**
- D6 commands "in any channel the relay reads" × D8 bridge channels that both machines' relays read: **broken**
- D6 `approve &lt;machine&gt;` × D1 own-name refusal: **broken** (part of the previous finding)
- D6 pin × D8 resync adoption (newest envelope message): **broken**
- D8 "no pin → hold" × D1 own current or former names: **broken**
- D4 retirement archive × D8 resync and delivery on the peer: **broken**
- D3 program-only verbs × D9 trusted stamp: held. The forgery bound is consistent with what the stamp grants.
- D2 orch-only post × D9 untrusted stamp: held. The untrusted stamp is reachable only from a peer that names a non-orch sender.
- D3/D7 outbox re-settle × D7 "never posted again": held within the record; whether the existing outbox clear interferes belongs to the existing-paths lens.
- D8 per-channel hold × D10 poll back-off ("a re-read held message is not new"): held
- D8 button-off hold × D10 relay lifetime: held
- D5 envelope × D1 parser; D5 line cap × D9 longest stamp: held
- D6 revoke × D8 held and pinned delivery: held
- D10 poll × `PATTERN-daemon-wakeups` constraint: held
- AC D4 "holds on one made by anyone else" and the refused-invite criterion × D7 five-refusal return: **broken**
- AC D7 × D7 five-refusal return: no criterion
- AC D11 (Review: `role-slack.md`) × D11: the criterion states something D11 does not

**Findings**

1. MEDIUM, design. Location: D3 × D10.
   A posted but unsettled bridge message can be returned to its sender as undelivered after Slack accepted it.
   - D3 guarantees re-settle only "before the hour", by the relay at its start.
   - D10 lets the relay stop entirely when every button of the hub is off.
   - So a post Slack accepted, whose settle was lost, sits with the relay down for more than an hour. Creel then returns it with its last hold reason or `bridge-unsettled`, and the sender gets a return notice for mail the peer received.
   - When the relay later restarts, D3 has it settle a message creel already returned. The record does not say what creel does with that.
   - Evidence: decision-record.md:57-59 (D3) and :162 (D10); `outboundDeadline = time.Hour` in internal/creelengine/outbound.go:24.

2. MEDIUM, design. Location: D6 × D8 (bridge channels).
   Operator commands typed in a bridge channel are handled by both machines' relays.
   - D6 handles `approve`, `revoke` and `status` "in any channel the relay reads, and only from the configured operator".
   - D8 has both ends read every bridge channel, and answers any other operator text as not delivered.
   - So one `approve osl-1046` typed in a bridge channel is also run by osl-1046's own relay, which pins its own app as a peer under its own name. Neither D6 nor D1 refuses approving this machine's own name.
   - `revoke` likewise acts on both machines, every command or plain operator line gets two answers, and the two "not delivered" replies conflict.
   - D8's "an `approve` written after a held message in the same channel" has the same double handling.
   - Evidence: decision-record.md:99 (D6), :135-136 (D8), :34 (D1 refuses only posts).

3. MEDIUM, design. Location: D8 resync × D6 pin.
   The resync adoption guard does not check who wrote the envelope it trusts.
   - D8 checks the channel's creator against this bot or an approved peer's bot. It then takes the local orch from "the newest message holding an envelope", read from any poster.
   - Any workspace member can join a public `bridge-` channel and post an envelope line naming another hub's orch. A hub on this machine then adopts the channel with the wrong local orch.
   - The pin that guards delivery (D6, D8's per-message bot and app checks) is not applied to adoption, and the bound "a channel no approved bot made is never read" does not cover it.
   - Evidence: decision-record.md:129 (resync rule), :137-139 (per-message human skip and pin check that adoption skips), :150 (bound).

4. LOW, consistency. Location: D4 × D7.
   A refused invite of the peer's bot is told both to hold and to return.
   - D4 holds the message "with Slack's error code".
   - D7 settles a message "Slack refuses with an error code five times" as returned.
   - An invite refusal is a Slack error code, so the record does not say whether it counts toward the five, or whether the hold outlives them until the hour.
   - The same ambiguity covers `bridge-name-taken`, and it carries into AC D4 versus D7.
   - Evidence: decision-record.md:79 (D4), :117 (D7 hold list), :119 (five refusals); internal/slackengine/outbound.go:356-370 counts every `*APIError` and `errChannelRefused` toward the cap.

5. LOW, consistency. Location: D4 × D7.
   "A listing of the workspace's channels made once per pair" does not hold while the message is held.
   - A message held on `bridge-name-taken` is retried at each hold check, and nothing caches the refused find.
   - Each retry's create then answers `name_taken` and runs `FindChannel` again, which pages every public channel of the workspace. That is up to once per bridge-poll wake, for as long as the hold lasts.
   - Evidence: decision-record.md:76 and :78; internal/slackengine/api.go:278-307 (`FindChannel` pages all channels); keeper.go:186.

6. LOW, design. Location: D8 × D1.
   A peer message whose `from` names this machine is held forever, with a notice asking for `approve &lt;this machine&gt;`.
   - D1 refuses this machine's current and former names only at post.
   - D8's "no pin → hold, notice asking for `approve &lt;machine&gt;`" applies to any `from` machine without a pin, this machine included.
   - Together with finding 2, that notice invites the operator to self-pin.
   - Such a message should be skipped, the way a `to` that is not the local orch is skipped.
   - Evidence: decision-record.md:34, :140, :141.

7. LOW, design. Location: D4 retirement × D8 resync and delivery.
   - When an orch retires, D4 archives its bridge channels.
   - The peer's next post finds the channel archived, drops it, and creates a new one under the same computed name (D4).
   - This machine's resync then adopts it with the retired orch as local end (D8), and delivers through the deliver verb to an orch that no longer exists.
   - The record states no outcome for mail to a retired local orch, and the peer's sender learns "delivered", since Scope excludes receipts.
   - Evidence: decision-record.md:81-82 (D4), :128-129 and :143 (D8).

8. LOW, consistency. Location: acceptance criteria × D7 and D11.
   - AC D7 has no criterion for D7's five-refusal return, which settles the message returned with the Slack code.
   - The Review criterion tagged D11 requires `contracts/orchrole/role-slack.md` content, but D11 does not state it; only Scope names "the orch role's Slack file".
   - Evidence: decision-record.md:119, :210-212, :225, :18.

Tool calls: none. Inherited context covered every pair.
