# Card 8 — chase-reply-forward

**Edit:**
- `internal/creelengine#Store.Reply`
- `internal/creelengine#Store.Forward`
- `internal/creelengine_test#TestScheduler_Chases`

**Extends:**
- `internal/creelengine/doc.go`

**Uses:**
- `internal/creelengine#isChaseable`
- `internal/creelengine_test#env`

**Intent:**
A chase is answered as the message it chases, and is never forwarded (D10).

`Reply` to a message whose `ChaseOf` is set takes the reply as a reply to the chased message: it is addressed to the chased message's sender, carries the chased message's id as `ReplyTo`, is validated as a reply to that sender, and so sets the chased message's `Replied` through the fold, which stops its chase clock.
It is accepted only from the chased message's current holder.
A sender that holds the chase but not the chased message is refused with "creel: message #<chase> chases #<id>, which <holder> holds now; way forward: the strand that holds #<id> answers it, as lym creel log --id <id> shows".
That branch is a fail-loud guard in `refusal-spec.md`'s sense, taking no reaching test: a chase goes only to the holder of a read message, and `Forward` refuses a read message, so no path moves the chased message away from the chase's holder today; the guard keeps the reply from being credited to a strand that does not hold the message should such a path appear (Shared Decision `former-holder-reply-guard`, and the decision record's Added after Discussion entry for review F5, which amends D10 and its Constraint so).

`Forward` of a message whose `ChaseOf` is set is refused before the receiver check with "creel: message #<id> is a chase and cannot be forwarded; way forward: answer it with lym creel post --reply <id> \"<text>\", which answers #<chased>", and nothing is journaled.
The input that reaches it is the holder forwarding the unread chase it holds.
D10 also names forwarding the chased message itself as a way forward; the refusal leaves that clause out, because the chased message is always read and `Store.Forward` refuses a read message, so the step could never succeed (Shared Decision `chase-forward-way-forward`, and the same Added after Discussion entry).

`Reply` to a `CREEL` message that is no chase, the sender notice of an `unanswered` outcome, is refused with "creel: message #<id> is a creel notice and takes no reply; way forward: post to the strand it names with lym creel post <strand> \"<text>\"", since the `CREEL` program holds no mailbox and such a reply would sit unread; the input that reaches it is the notified sender replying to that notice.

`TestScheduler_Chases` gains rows: after chase 1, the holder's `Reply` to the chase id posts a reply to the chased message's sender with `ReplyTo` the chased id, no chase 2 follows, and the outcome is never `unanswered`; the holder's `Forward` of the unread chase is refused with the message above and leaves the chase with its holder; after an `unanswered` outcome, the sender's `Reply` to its `CREEL` notice is refused naming `lym creel post <strand>` and journals nothing.
`internal/creelengine/doc.go`'s reply discipline section states both rules.

**ImpactSummary:** Reply gains a chase branch and a creel-notice refusal and Forward a chase refusal; replies and forwards of every other message are unchanged.
