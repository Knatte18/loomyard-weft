```yaml
lens: walkthrough
outcome: reported
```

Task: walk each Decision through one concrete case, using the lens's procedure.

**Walkthrough list**
- D1, a post from `neve` `ly:orch` to `osl-1046::ly:orch`, and one to the machine's own `neve::ly:orch`: purpose met. A former name that fails `CheckName` takes another path (NIT 7).
- D2, a beam orch posting to `osl-1046::ly:orch`, and a pair's orch posting the same: purpose met.
- D3, a post that is journaled, taken, posted and recorded in the outbox, after which the sending orch's button, the hub's last one on, goes off before settle: purpose missed (finding 1).
- D4, both sides sending at once: neve creates the channel and osl gets `name_taken`, adopts the channel and joins it. Purpose missed: neve's first message is skipped (finding 2).
- D5, the envelope `[lym-bridge] from=neve::ly:orch to=osl-1046::ly:orch id=42` through `slackEscape` going out and `inboundText` coming back: purpose met.
- D6, the operator typing `approve osl-1046 A0ABC123` in neve's orch channel: purpose at risk (finding 4). The same command typed in a bridge channel: purpose missed (finding 5).
- D7, a message held for `peer-unapproved` on hub `ly`, with the approval made in a channel of neve's hub `ab`: purpose not shown (finding 6).
- D8, first contact from osl to neve with neve's operator not yet having approved osl: purpose missed (finding 3). A pinned reply with a valid reply-to: purpose met.
- D9, an inbound message from `osl-1046::ly:orch` with role orch, then one from a non-orch sender: purpose met.
- D10, an idle relay at the 60-second ceiling when the peer posts and no Socket Mode event arrives: met, since the next poll reads the message.
- D11, the scopes of the approve calls checked against the manifest, and the `lym-&lt;machine&gt;` bot name checked against its `display_name`: purpose met.

**Findings**

1. MEDIUM · design · D3 (and D10). A settle lost when the relay stops is not made again before the hour, so mail that was delivered is reported as returned.
   - Case: the relay posts the message and writes the outbox. The orch's button, the hub's last one on, then goes off, so the relay stops (D10: "The relay runs while some orch of the hub has its button on").
   - Nothing settles the message. At its hour creel returns it with `bridge-unsettled` and sends the sender a return notice, even though the peer received it.
   - D3 never says what creel does with a settle-delivered that arrives for a message already returned.
   - Evidence: `_lym/discussion/decision-record.md:57-59` and `:162`; `internal/creelengine/outbound.go:24` (`outboundDeadline = time.Hour`).
2. MEDIUM · design · D4 and D8. No Decision says where the read cursor of an adopted bridge channel starts. The existing adopt starts it at the channel's newest message, which drops the first message.
   - The keeper's adopt sets `Cursor: newest, ThreadFloor: newest`.
   - Case: both sides send at once. osl adopts neve's channel after neve has already posted, so osl's cursor starts after neve's message and that message is never delivered.
   - A channel added by a resync has the same gap. The newest envelope message is the one that triggered the add, and a cursor at "newest" skips it.
   - This misses Q6's stated purpose, "no lost first message".
   - Evidence: `internal/slackengine/keeper.go:226-233`; `decision-record.md:76-77` and `:128-129`; `support-log.md:86`.
3. MEDIUM · design · D8. On first contact, the hold-and-notify path for an unapproved peer cannot be reached.
   - A resync adds a `bridge-` channel only when "its creator is this bot or an approved peer's bot user".
   - On first contact the peer's bot made the channel and the peer is not yet approved. The channel is never mapped, so the bullet "When the `from` machine has no pin, the message is held, and the relay posts one notice … asking for `approve &lt;machine&gt;`" never runs.
   - The receiving operator gets no notice anywhere. The mail sits unread in Slack until they approve the peer unprompted, and the next resync then adopts the channel (see finding 2 for what happens next).
   - The D8 criterion "a message from a machine with no pin holds the delivery cursor and posts one notice" assumes a channel that is already mapped.
   - Evidence: `decision-record.md:129`, `:140` and `:216`.
4. LOW · design · D6. `approve &lt;machine&gt; &lt;app id&gt;` is said to be "handled like `away`, `back` and `status`". That handling lower-cases the whole text before matching, which would lower-case the app id.
   - Slack app ids are upper case, such as `A0…`, so a lower-cased id matches no app.
   - Evidence: `internal/slackengine/inbound.go:605-606`; `decision-record.md:99-102`.
5. LOW · design · D6 and D8. Both machines' relays read a bridge channel, and D8 handles the operator's messages there as commands. So one `approve`, `revoke` or `status` typed in a bridge channel is handled by both machines, and each answers.
   - Case: in `bridge-neve-ly-orch--osl-1046-ly-orch` the operator types `approve osl-1046`. neve pins osl as intended.
   - osl also handles the same text. It finds its own bot `lym-osl-1046` and pins its own app as a peer.
   - Text that is not a command is answered as "not delivered" twice.
   - Evidence: `decision-record.md:99` ("in any channel the relay reads") and `:135-136`.
6. LOW · design · D7. "A held message is posted once its hold clears" names nothing that wakes the relay when an approval clears the hold.
   - The approved-peer record is machine-wide in the credential directory. An `approve` handled by another hub's relay writes it, and the relay holding the mail watches only its inbox and forwarding file.
   - Evidence: `decision-record.md:96` and `:120`; `pattern/PATTERN-daemon-wakeups.md:97`.
7. NIT · consistency · D1. A former machine name that fails `CheckName` cannot be refused naming the bare name.
   - `machinename.Names` keeps former names "even when it fails the name rule". D1's parser rejects such a machine part first, so a post to one gets the parser's error instead of the way forward D1 promises.
   - Evidence: `internal/machinename/machinename.go:79-83`; `decision-record.md:33-34`.

**Tool calls**
- Read the manifest and bot naming in `docs/slack-app.md`, to check that the bot name D6 searches for matches the manifest's `display_name`. My context had only the scope lines.
