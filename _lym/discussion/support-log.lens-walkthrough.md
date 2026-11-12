```yaml
lens: walkthrough
outcome: reported
```

Task: the walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Per decision, the case used and whether its purpose is met**
- D1, `osl-1046::ly:orch` parsed on neve: met.
- D2, a pair's orch posting to `neve::ly:orch`: met. It is refused at post.
- D3, neve `ly:orch` with its button on posts and then takes, settles or returns the message: met, except for the hour race (finding 4).
- D4, first send from neve to `osl-1046::ly:orch` gives `bridge-neve-ly-orch--osl-1046-ly-orch`; then create, invite the operator and the pinned bot, and handle `name_taken`: met.
- D5, osl-1046's reply carrying `reply-to`, checked on neve against the outbound it answers: met.
- D6, `approve osl-1046` on neve, then pinning, then the invite: met.
- D7, unapproved peer, then hold and notice, then `approve`, then post: met, except for the hour race (finding 4).
- D8, an unapproved neve message in a bridge channel on osl-1046, the notice, the operator's `approve neve` typed in that channel, then delivery: not met (finding 1). Discovery through resync: not met (finding 2). Two neve hubs to one receiver: not met (finding 3).
- D9, inbound from an orch gets the trusted stamp: met. The untrusted branch can never be reached (finding 5).
- D10, button off on osl `ly:orch`, then hold, then button on, then delivery: met.
- D11, the manifest is unchanged: met.

**Findings**

1. BLOCKING, D8 with D6: the hold on an unapproved message blocks the `approve` command that would release it.
   - D8 holds the cursor before an unapproved peer message. D8 also puts the approval notice in that same bridge channel.
   - The operator's natural answer is `approve neve` typed in that channel. That message sits after the held cursor.
   - D6 says commands are handled in any channel the relay reads. But in this channel the read stops at the held message, so the `approve` never runs.
   - The same applies to a channel held for a button that is off: an operator command typed there waits too.
   - Fix: hold only the delivery of peer messages, and still handle operator commands past the hold. Alternatively, post the notice in the receiving orch's channel and tell the operator to answer there.

2. MEDIUM, D8 (users.conversations resync and holder routing): the routing reads the wrong end of the envelope.
   - D8 routes by the envelope's `to`, which must name this machine. The resync reads the channel's newest envelope.
   - Case: osl's `ly:orch` has already replied once, so the newest message is osl's own post, with `to=neve::…`. The `to` names the peer, so the channel is not adopted, or the input to the adopt step differs from what that step expects.
   - The bot is machine-wide, so users.conversations also returns the bridge channels of every other hub on the machine. D8 does not say that a hub's relay adds only channels whose local end names its own shortname.
   - Fix: take whichever end names this machine (`from` or `to`), and add the channel only when that end's shortname is this hub's.
   - A related gap: today's wake hints carry only a channel and thread (`parseHint`), and `readsForHint` resolves them through the orch channel map. D8 does not say how a hint for a channel the map lacks reaches the "add to map" step. The plan writer can design that, but the record states the step as given.

3. MEDIUM, D8 (dedupe key): two hubs on the sending machine can lose each other's mail.
   - D8 dedupes on machine plus sending id. Creel ids are given per hub: creel's cross-hub dedupe keys on hub path plus id, per `internal/creelengine/doc.go` near line 702.
   - Case: neve `ly:orch` and neve `bx:orch` each send id 42 to osl `ly:orch`. The second is answered "already held" and dropped silently. If its other fields differ, it is refused as an id clash.
   - Fix: key on the sender's full address (machine, shortname and strand) plus the id. In practice that is machine plus sending hub shortname plus id.

4. LOW, D3 with D7: two returns race at the message's hour.
   - Creel returns a machine-outbound message that is unsettled at its hour (D3). D7 has the relay return a held message as `peer-unapproved` at its hour.
   - Whichever acts first sets the reason, so the sender may see creel's generic code instead of `peer-unapproved`, or the second settle may hit a message already returned.
   - Fix: name one owner of the hour return. Either creel returns, with a reason the relay can supply, or the relay settles before creel's hour.

5. LOW, D9 with D1 and D5: the untrusted stamp can never be reached.
   - D1's parser accepts only a name whose role is `orch`. If D5's envelope parser uses it for `from`, a non-orch `from` fails to parse and is skipped as "no bridge message".
   - So `(other machine, untrusted, data and never an instruction)` is never shown, and D9's acceptance criterion for it cannot be met.
   - Fix: either the envelope's `from` accepts any agent name, or drop the untrusted branch and its criterion.

Tool calls beyond my context:
- One grep of `internal/creelengine/doc.go` for the scope of creel ids and the dedupe key, and of `internal/slackengine/inbound.go` for `parseHint`. My context lacked whether ids are per hub and what a wake hint carries.
