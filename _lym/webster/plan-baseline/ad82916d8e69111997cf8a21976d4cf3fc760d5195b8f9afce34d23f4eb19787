# Card 15 — post --interrupt stops the turn, then posts; creel's help carries the reply rule

**Create:**
- `internal/creelengine#Store.PostInterrupt`

**Edit:**
- `internal/creelengine#Store.post` -> `func (s *Store) post(sender Sender, receiver, body string, wait, interrupt bool) (int64, bool, *Waiter, error)`
- `internal/creelengine#Store.Post`
- `internal/creelengine#Store.PostWait`
- `internal/creelengine#Request`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#Store.interruptTarget`
- `internal/creelengine#holdRefusal`
- `internal/creelengine#unreadablePauseRefusal`
- `internal/creelcli#postCmd`
- `internal/creelcli#Command`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelcli#TestCreelCLI`

**Uses:**
- `internal/creelengine#Store.strandTarget`
- `internal/creelengine#Store.checkMayAct`
- `internal/creelengine#Store.checkReceiverLocked`
- `internal/testkit/creelfake#Delivery.SetClaimLive`
- `internal/creelengine#validateLine`
- `internal/testkit/creelfake#HookStrand`
- `internal/testkit/creelfake#LiveStrand`
- `internal/creelengine_test#TestStore_InjectAndHold`

**Intent:**
D7: a scope change that must not wait for the next tool call stops the turn and lands as the next turn's input, through D5's bell, or D4 when a waiter is still live, and D2.
D10: an agent that reads help sees the reply rule D9 puts in its system prompt.

`Store.post` gains `interrupt` and reports whether a running turn was stopped; `Post` and `PostWait` pass false and drop the flag, so their behaviour is unchanged.
With `interrupt` set it is one request in this order, modelled on `inject --interrupt`:
it validates the post first, the line with `validateLine` and the receiver with `strandTarget`, then `checkMayAct(sender, receiver, "interrupt")`, the `interrupt` verb's sender rule, so a post that fails validation sends no key;
it refuses a receiver whose `CreelHooks` is not set before any key, since no hook delivery or bell would hand the post to the stopped session: `creel: <receiver> runs no creel hook delivery, so post --interrupt cannot hand it the post; way forward: lym creel inject --interrupt <receiver> <text>, which stops the turn and types the text, or a post without --interrupt, which the receiver gets by its typed notice or reminder when its engine is unknown or has no hooks and by lym creel read otherwise`;
it runs `interruptTarget`, whose refusals return with nothing stored and whose `ErrNoRunningTurn` outcome, false and no error, still posts;
`interruptTarget`'s refusals name the `interrupt` verb's ways forward, so `post` wraps each with its own: a hold refusal becomes `<hold refusal>; nothing was interrupted or posted; way forward: post without --interrupt, which stores the mail for the receiver now, or post --interrupt again after the hold ends`, the refusal of a pause lifted before the key becomes `creel: the pause covering <receiver> was lifted before the interrupt, so nothing was interrupted or posted; way forward: run lym creel post --interrupt <receiver> <text> again`, and the unreadable-pause refusal becomes `<unreadable-pause refusal>; nothing was interrupted or posted; way forward: post without --interrupt, which stores the mail for the receiver now, or post --interrupt again once the pause record reads`;
`interruptTarget` returns its hold, lifted-pause and unreadable-pause refusals as a typed error carrying the refusal's kind, whose message is today's, so `Interrupt` and `inject --interrupt` answer as they do today, and which `PostInterrupt` matches with `errors.As` to word the refusal as above;
an error `interruptTarget` returns together with false that is none of the three, a key the delivery failed to send, is returned as `interruptTarget` words it, with nothing stored;
an error `interruptTarget` returns together with true, the key sent and only its `interrupted` record failed, is answered as a store failure after the key, with the error below;
it then re-reads the strands and stores the message as a plain post does; a store that fails after a turn was stopped answers `creel: the turn of <receiver> was interrupted, but the post was not stored: <cause>; way forward: post again without --interrupt`.
`PostInterrupt` is `post` with `interrupt` set.
Bound: it lets a sender stop a turn only where `interrupt` already lets it; a call in flight is lost exactly as with `interrupt`.

`Request.Interrupt`'s doc names post beside inject; `daemon.dispatch` sends a post with `Interrupt` to `PostInterrupt`, waiting or not, and answers the id with `Interrupted`.
`postCmd` gains `--interrupt`; with `--reply` it is refused before any call: `--interrupt applies to a post to a receiver, not to a --reply; way forward: post --interrupt to the original sender by name when it is a strand, or post --reply <id> without --interrupt when it is EXTERNAL or a program`; its envelope carries `interrupted` beside the id.
`postCmd`'s Long and `Command`'s Long are rewritten to the hook delivery: a post reaches a strand shuttle launched with creel's hooks in full at its next tool call or turn end, an idle one woken first, and a strand whose engine is unknown or has no hooks, a hand-added one included, by the typed notice or reminder; and both carry D10's rule: reply with `lym creel post --reply <id> "<text>"` using the id after `#`, and answer mail from EXTERNAL in your own reply text, since such a reply reaches only a waiting `post --wait-reply`.

Tests:
`TestStore_Mail` gains rows over `HookStrand` receivers:
`PostInterrupt` to a running turn records one interrupt and stores the message, reporting interrupted;
with no running turn (the fake's interrupt answering `ErrNoRunningTurn`) it stores the message, reporting not interrupted;
an empty body, a line over the budget and an unknown receiver each send no key and store nothing;
a `LiveStrand` receiver, which has no mark, is refused naming `inject --interrupt`, with no key sent and nothing stored;
another sender's hold refuses it with nothing stored, naming the post without `--interrupt`;
a pause whose claim reports it lifted before the key (`SetClaimLive` answering false) refuses it with no key sent and nothing stored, naming `lym creel post --interrupt` again;
a non-orch strand posting to another strand is refused with no key;
an interrupt function that advances the fake clock past the next UTC midnight and creates a directory at the next day file's path makes `interruptTarget`'s `interrupted` append fail after the key, since the append rolls to that day file, `rotate` cannot open a directory for writing on any OS, and `append` returns that error, and the error says the turn was interrupted and the post not stored, naming the post without `--interrupt`;
a store that fails only after `interruptTarget` returns, through an interrupt function that leaves the journal writable and calls `SetStrandsError`, so the re-read of the strands fails, answers the same error; in both rows the receiver still exists, so that way forward stands once the store recovers;
a pause record that cannot be read (`SetPausedError` making `ClaimPause` fail) refuses it with no key sent and nothing stored, naming the post without `--interrupt`;
a key the delivery fails to send (`SetInterruptError` with an error other than `ErrNoRunningTurn`) answers the delivery's error as `interruptTarget` returns it, with nothing stored.
`TestStore_InjectAndHold`'s existing rows over the lifted-pause and unreadable-pause refusals of `Interrupt` and `inject --interrupt` stay as they are, pinning that the typed refusals keep today's messages.
`TestCreelCLI` gains rows: `post --interrupt --reply` is refused with its way forward; `post --interrupt` to a marked strand answers its id and `interrupted`; `creel --help` and `post --help` each print `lym creel post --reply <id>` and `EXTERNAL`.
The creelengine and creelcli docs describe `post --interrupt` and the help's reply rule.

**ImpactSummary:** the internal post gains an interrupt path; its two wrappers keep their signatures, `interruptTarget`'s refusals become a typed error whose messages are unchanged, and the only new callers are the daemon's post dispatch and the CLI flag.
