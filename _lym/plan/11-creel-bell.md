# Card 11 — A typed fixed bell wakes an idle hook strand no waiter reaches

**Create:**
- `plan:internal/creelengine#BellLine` -> `const BellLine = "[creel] Mail is waiting for you. End this turn now; creel hands the mail to you as the turn ends. Do not run lym creel read for it."`
- `plan:internal/creelengine#bellGrace` -> `const bellGrace = 5 * time.Second`
- `plan:internal/creelengine#ringAnswerWindow` -> `const ringAnswerWindow = 30 * time.Second`
- `plan:internal/creelengine#worker.typeBell` -> `func (w *worker) typeBell(strand Strand, ids []int64) bool`

**Edit:**
- `internal/creelengine#wakeMark`
- `internal/creelengine#worker.wakeHookStrand`
- `internal/creelengine_test#TestScheduler_HookWake`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Waiter.Cancel`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine/doc.go`

**Uses:**
- `internal/creelengine#worker.idle`
- `internal/creelengine#Delivery`
- `internal/testkit/creelfake#Delivery.SetStall`
- `internal/testkit/creelfake#Delivery.Send`

**Intent:**
D5: a user interrupt fires no Stop, so after `interrupt` or `post --interrupt` no new waiter starts, and a strand marked `CreelHooks` can sit idle with mail no waiter reaches; the delivery worker then types D4's fixed bell line, which carries no sender text, so no post is typed.
`BellLine` is D4's line verbatim; card 13's waiter writes the same constant to its standard error on a ring.

`wakeMark` gains whether a bell was typed for its set and whether its ring was answered.
A ring is answered by a claim of the strand that hands out at least one message, which `ClaimAtBoundary` records on the strand's mark, or by a probe reading a turn in progress after the ring.
`Store` gains the clock time its last hook waiter of each strand ended, which `Waiter.Cancel` records for a `waitHook` key, and the time the store opened; a strand whose waiter never registered counts from that opening.

`wakeHookStrand`, when it does not ring, types the bell through `typeBell` when the claimable set is non-empty, no bell was typed for that set, and one of two rules holds.
For a strand with no unanswered ring standing, no waiter is live and none has been since `bellGrace` (5 seconds after the later of the store's opening and the strand's last waiter ending), which keeps the bell out of the moment between a Stop and its waiter's connection.
For a strand whose last ring is unanswered, the grace rule does not apply, since a ring answers and so ends the waiter it rings; the bell waits until the ring has gone unanswered for `ringAnswerWindow` (30 seconds), as D4 says.
A newer set clears only the mark's bell record, so its bell may be typed once more; it keeps the ring time and whether that ring was answered, which card 10's hold and the window rest on; an unchanged set never gets a second bell.
The bell's idle gate is the guarded send's own `Delivery.Idle`, as D5 prescribes; a woken turn whose hook reading reads ready is answered by its first claim, its first tool call or its turn end, well inside the window, so the window, not the idle gate, keeps the bell out of a woken turn.
`typeBell` types `BellLine` through the guarded `Delivery.Send` under every rule a typed line has: it asks `worker.idle` first, so a draft, a too-short pane or a missing input box holds it and feeds the stall journal; a nil answer or `ErrNotLanded` records the bell for the set, `ErrNotLanded` never retyped; `ErrBusy` records nothing and the next probe tries again.
Pause and hold apply as for the ring: a paused strand is never probed, and the set holds only the hold owner's mail.
Only a strand marked `CreelHooks` gets a bell: a pane-mail strand has its notice and reminder, and a strand with neither field, a Claude strand without the mark, reads its mail with `lym creel read` (D6).
Bound: one fixed line per claimable set, only into a strand marked `CreelHooks`; a waiter whose session died rings into nothing once and the window hands the set to the bell; a restarted daemon may type the bell once more for a set it already woke.

`TestScheduler_HookWake` gains rows:
an idle hook strand with no waiter gets no bell before the grace and `BellLine` once after it, and an unchanged set over later probes gets no second;
a new post after the bell types it once more;
a draft stall holds the bell until the stall clears, and a busy send types it on the next probe;
a send answering `ErrNotLanded` records the bell, and later probes of the same set type nothing more;
a ring unanswered for the window is followed by the bell, with no bell between the grace's 5 seconds and the window's 30; a ring answered by a claim 20 seconds after it is not followed by the bell; and a ring answered by a probe reading a running turn (`SetTurn` running true) after it is not followed by the bell either;
a waiter that ends without a ring starts the grace again from its end;
under another sender's hold, mail only the hold mutes types no bell, and the owner's mail does;
a `LiveStrand` and a strand with neither field never get the bell;
a paused hook strand gets none.
The doc's idle-wake section gains the bell.

**ImpactSummary:** the worker's hook-strand step gains the bell and the store tracks waiter ends and ring answers; no verb or caller outside creelengine changes.
