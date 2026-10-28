# Card 15 — post --interrupt stops the turn, then posts; creel's help carries the reply rule

**Create:**
- `plan:internal/creelengine#Store.PostInterrupt` -> `func (s *Store) PostInterrupt(sender Sender, receiver, body string, wait bool) (int64, bool, *Waiter, error)`

**Edit:**
- `internal/creelengine#Store.post` -> `func (s *Store) post(sender Sender, receiver, body string, wait, interrupt bool) (int64, bool, *Waiter, error)`
- `internal/creelengine#Store.Post`
- `internal/creelengine#Store.PostWait`
- `internal/creelengine#Request`
- `internal/creelengine#daemon.dispatch`
- `internal/creelcli#postCmd`
- `internal/creelcli#Command`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelcli#TestCreelCLI`

**Uses:**
- `internal/creelengine#Store.interruptTarget`
- `internal/creelengine#Store.strandTarget`
- `internal/creelengine#Store.checkMayAct`
- `internal/creelengine#Store.checkReceiverLocked`
- `internal/testkit/creelfake#Delivery.SetClaimLive`
- `plan:internal/creelengine#validateLine`
- `plan:internal/testkit/creelfake#HookStrand`
- `internal/testkit/creelfake#LiveStrand`

**Intent:**
D7: a scope change that must not wait for the next tool call stops the turn and lands as the next turn's input, through D5's bell, or D4 when a waiter is still live, and D2.
D10: an agent that reads help sees the reply rule D9 puts in its system prompt.

`Store.post` gains `interrupt` and reports whether a running turn was stopped; `Post` and `PostWait` pass false and drop the flag, so their behaviour is unchanged.
With `interrupt` set it is one request in this order, modelled on `inject --interrupt`:
it validates the post first, the line with `validateLine` and the receiver with `strandTarget`, then `checkMayAct(sender, receiver, "interrupt")`, the `interrupt` verb's sender rule, so a post that fails validation sends no key;
it refuses a receiver whose `CreelHooks` is not set before any key, since no hook delivery or bell would hand the post to the stopped session: `creel: <receiver> runs no creel hook delivery, so post --interrupt cannot hand it the post; way forward: lym creel inject --interrupt <receiver> <text>, which stops the turn and types the text, or a post without --interrupt, which the receiver gets by its typed notice when its engine has no hooks and by lym creel read otherwise`;
it runs `interruptTarget`, whose refusals return with nothing stored and whose `ErrNoRunningTurn` outcome, false and no error, still posts;
`interruptTarget`'s refusals name the `interrupt` verb's ways forward, so `post` wraps each with its own: a hold refusal becomes `<hold refusal>; nothing was interrupted or posted; way forward: post without --interrupt, which stores the mail for the receiver now, or post --interrupt again after the hold ends`, and the refusal of a pause lifted before the key becomes `creel: the pause covering <receiver> was lifted before the interrupt, so nothing was interrupted or posted; way forward: run lym creel post --interrupt <receiver> <text> again`;
it then re-reads the strands and stores the message as a plain post does; a store that fails after a turn was stopped answers `creel: the turn of <receiver> was interrupted, but the post was not stored: <cause>; way forward: post again without --interrupt`.
`PostInterrupt` is `post` with `interrupt` set.
Bound: it lets a sender stop a turn only where `interrupt` already lets it; a call in flight is lost exactly as with `interrupt`.

`Request.Interrupt`'s doc names post beside inject; `daemon.dispatch` sends a post with `Interrupt` to `PostInterrupt`, waiting or not, and answers the id with `Interrupted`.
`postCmd` gains `--interrupt`; with `--reply` it is refused before any call: `--interrupt applies to a post to a receiver, not to a --reply; way forward: post --interrupt to the original sender by name when it is a strand, or post --reply <id> without --interrupt when it is EXTERNAL or a program`; its envelope carries `interrupted` beside the id.
`postCmd`'s Long and `Command`'s Long are rewritten to the hook delivery: a post reaches a Claude strand in full at its next tool call or turn end, an idle one woken first, a strand whose engine has no hooks by the typed notice or reminder; and both carry D10's rule: reply with `lym creel post --reply <id> "<text>"` using the id after `#`, and answer mail from EXTERNAL in your own reply text, since such a reply reaches only a waiting `post --wait-reply`.

Tests:
`TestStore_Mail` gains rows over `HookStrand` receivers:
`PostInterrupt` to a running turn records one interrupt and stores the message, reporting interrupted;
with no running turn (the fake's interrupt answering `ErrNoRunningTurn`) it stores the message, reporting not interrupted;
an empty body, a line over the budget and an unknown receiver each send no key and store nothing;
a `LiveStrand` receiver, which has no mark, is refused naming `inject --interrupt`, with no key sent and nothing stored;
another sender's hold refuses it with nothing stored, naming the post without `--interrupt`;
a pause whose claim reports it lifted before the key (`SetClaimLive` answering false) refuses it with no key sent and nothing stored, naming `lym creel post --interrupt` again;
a non-orch strand posting to another strand is refused with no key;
an interrupt function that closes the store's journal makes the store fail after the key, and the error says the turn was interrupted and the post not stored, naming the post without `--interrupt`; the receiver still exists, so that way forward stands once the journal is writable again.
`TestCreelCLI` gains rows: `post --interrupt --reply` is refused with its way forward; `post --interrupt` to a marked strand answers its id and `interrupted`; `creel --help` and `post --help` each print `lym creel post --reply <id>` and `EXTERNAL`.
The creelengine and creelcli docs describe `post --interrupt` and the help's reply rule.

**ImpactSummary:** the internal post gains an interrupt path; its two wrappers keep their signatures, and the only new callers are the daemon's post dispatch and the CLI flag.
