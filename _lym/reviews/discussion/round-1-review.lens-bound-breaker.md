```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens on the slack-machine-bridge decision record.

**Claims attacked**

- D1, refusal of own current or former name at post: held at post. **Broken** against the parser: `machinename.Names` keeps former names that fail the name rule, but D1's parser requires `CheckName`. A rule-failing former name is refused by the parser and never reaches the "naming the bare name" refusal.
- D2, the bound that opens only the orch-to-orch edge: held.
- D3, the claim that a program claim is accepted only from an orch pane or a process in no pane: **broken off Linux**. On other platforms identity comes from the environment, so a non-orch strand that unsets `LYM_STRAND_NAME` can claim `:SLACK`.
- D3, the claim that a lost settle "is made again before the hour": **broken**. The relay runs only while some orch's button is on, so a relay can stop between posting and settling for longer than the hour.
- D3, the claim that the take verb is the only reader and per-hub workers skip machine-outbound mail: held as a stated rule.
- D4, the bound that sanitized ends are not injective and the creator and envelope checks keep the wrong pair's mail out: held for delivery.
- D4, the claim that `FindChannel` is "a listing made once per pair": **broken**. A message held on `bridge-name-taken` is retried on each bridge poll, and each retry is a create, a `name_taken` answer and a full paged listing.
- D4, the claim that a refused invite holds the message, set against D7's five-refusal return: **conflict**, see the findings.
- D6, the bound that only the operator's account can approve: **broken in practice**. Both machines' relays read a bridge channel and both handle the same operator command.
- D6, the claim that "an approved app is trusted to name its sender": held for the machine part through D8's pin check.
- D8, the resync adoption bound that only an approved bot's channel is ever read: **broken**. The creator check covers who made the channel, but not who wrote the newest envelope, and anyone in the workspace can post in a public channel.
- D8, the routing edge that turns an unmapped-channel event into a resync hint for every hub: **unbounded**.
- D8, the bound that a held message is delivered only by an approval or the button: held. It is also broken for a peer that claims this machine's own name, see the findings.
- D8, the dedupe on the full sender address and sending id: held.
- D10, the claim that the poll alone carries the mail: held.
- D11, the claim that the relay already calls create, invite, join and archive and that `lym slack login` calls bots.info: held (`api.go:263-335`, `slackcli/cli.go:282`).

**Findings**

1. **MEDIUM, design. D3 (creel owns the hour; the relay re-settles from its outbox).**
   - Counterexample: the relay posts a bridge message and records it in its outbox. It crashes before settling, or it exits because the last orch's button went off.
   - It stays down for more than an hour. Creel then returns a message that Slack accepted and the peer delivered, and posts the return notice to the sender.
   - On restart, the relay settles delivered a message creel has already returned. The record does not say what creel answers to that.
   - Evidence: `internal/slackengine/start.go:113` (no relay while no button is on); `relay.go:495`; `internal/creelengine/outbound.go:24` (`outboundDeadline` is one hour).

2. **MEDIUM, design. D8 (resync adoption picks its local orch from the newest message holding an envelope).**
   - Bridge channels are public, so any workspace member can join one and post a fake `[lym-bridge] … to=&lt;this-machine&gt;::&lt;other-hub&gt;:orch` line.
   - The creator check still passes, so the other hub's relay adopts the channel under the wrong local orch.
   - The envelope read should count only messages from this bot or the pinned app.
   - Evidence: `internal/slackengine/api.go:263` (channels are created with `is_private=false`).

3. **MEDIUM, design. D8 (an unmapped-channel event becomes a resync hint for every hub).**
   - The bot gets `message.channels` events from every public channel it belongs to, and any member can invite it into one.
   - Each message there sends a resync hint to every hub. Today a resync hint means a full read of every open orch channel with its thread scan, and D8 adds a `users.conversations` listing on top.
   - Only the one-second wake floor bounds this, which risks Slack rate limits. The record states no bound for this edge.
   - Evidence: `internal/slackengine/inbound.go:358-365` (`readsForHint` for a resync); `holder.go:329-355` (`route`).

4. **MEDIUM, design. D6 with D8 (operator commands are handled in any channel the relay reads, and both machines' relays read a bridge channel).**
   - When the operator types `approve osl-1046` in a bridge channel, the relay on osl-1046 handles it too. It can pin its own app as a peer under its own name, since D6 never refuses this machine's own name.
   - Both relays also answer `status`, every non-command text and `revoke`, so a `revoke` acts on both machines at once.
   - Evidence: `internal/slackengine/inbound.go:599-613` (commands go only through the operator check, not a channel kind).

5. **LOW, consistency. D4 versus D7 (a refused invite holds the message; a message Slack refuses five times is returned).**
   - Neither decision says whether a refused invite or a `bridge-name-taken` refusal counts toward the five.
   - Today `refused` counts a keeper's `errChannelRefused` toward `refusalCap`. Reused here, a refused invite would return the message after five tries instead of holding it.
   - Evidence: `internal/slackengine/outbound.go:350-370`.

6. **LOW, consistency. D4 (`name_taken` lists the workspace's channels "once per pair").**
   - The held `bridge-name-taken` path repeats the paged `conversations.list` on every retry, up to once per bridge-poll ceiling. The "once per pair" claim is false on that path.
   - Evidence: `internal/slackengine/api.go:276-308`.

7. **LOW, design. D3 bound (the program claim is accepted from an orch pane and from a process in no pane).**
   - This holds only on Linux. Elsewhere `resolveFromEnvironment` accepts a program claim from any process, a non-orch strand's pane included, so such a strand can forge `(other machine, trusted orch)` mail.
   - Evidence: `internal/creelengine/identity.go:65-66,121-129`.

8. **LOW, design. D8 (a `from` machine with no pin is held with an `approve &lt;machine&gt;` notice).**
   - An approved peer's app can send `from=&lt;this machine&gt;::ly:orch`. This machine has no pin, so the message is held and the notice asks the operator to approve this machine itself.
   - Combined with finding 4, the operator can be steered into pinning a foreign app under this machine's name.
   - The record should skip a `from` that names this machine's current or a former name.
   - Evidence: `internal/machinename/machinename.go:83`.

9. **NIT, consistency. D1 (a post to this machine's former name is refused naming the bare name).**
   - `Names` keeps a former name that fails `CheckName`, and D1's parser refuses that machine part first. The post is refused with the parser's message instead of the stated way forward.
   - Evidence: `internal/machinename/machinename.go:79-82,90`.

Tool calls:
- grep of `internal/slackengine/start.go`, `relay.go` and `forwarding.go` for when the relay stops. The manifest did not cover the relay's lifecycle once no button is on.
