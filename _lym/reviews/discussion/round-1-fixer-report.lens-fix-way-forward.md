```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: check that every refusal, error, next step, row or test the fix added can be reached from the state it names and leads to progress.

**MEDIUM: a deliver still in flight when its message returns leads to a second return**
- Changed text: D3 step 5, "still pending, with no remote delivery and no return journaled, and `no` otherwise", together with the new D3 criterion "a confirm for a message already returned undelivered answers `no`".
- What goes wrong: D2's hour cap can return a message while a `deliver` for it is still in flight, since that call has a 30-second timeout. The confirm then answers `no`, so the receiver answers `refused`. D2 says "`refused` returns the message undelivered at once". Read as written, the message returns a second time: a second return is journaled and a second `CREEL` notice goes to the sender. Nothing tells the outbound worker to drop a `refused` answer for a message that has already returned.
- Evidence: D2 bullet "`refused` returns the message undelivered at once" against the new D3 step 5 and its acceptance row. Before the fix, a returned message still confirmed `yes`, so this refusal could not happen.
- Proposed change: in D2, state that a `refused` answer for a message already returned changes nothing.

**LOW: an older receiving binary is retried for an hour and the sender is never told why**
- Changed text: the D3 bullet "Any answer to `deliver` other than `accepted` or `refused`, an older binary's refusal of an unknown verb included, is a transient refusal the sending daemon retries (D2)."
- What goes wrong: when the receiving hub runs an older binary, every retry fails the same way. The message returns after the hour cap, and D2's `CREEL` notice gives "the reason" without saying what that reason is in this case. The sender learns only that the message returned, not that the receiving hub needs a binary carrying this task.
- Evidence: D2 bullet "sends the sender a `&lt;shortname&gt;:CREEL` notice naming the receiver, the reason and `lym creel log --id &lt;id&gt;`". Neither D2 nor D3 says that the last transient answer becomes the reason.
- Proposed change: state that the notice's reason for an hour-cap return is the last transient answer, so an unknown-verb refusal shows up there.

I made no further tool calls.
