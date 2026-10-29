```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, criteria and cross-references it touches, and look for twins left unchanged.

**MEDIUM: D3 now also wakes with the bell after a refused or lost acknowledgement, and the record does not say so**
- Changed text: D3, "Once the acknowledgement is written, `hook-wait` exits with the wake code whether the daemon confirms it, refuses it on a failed journal write, or the confirmation is lost."
- Summary: D3 now exits with the wake code even when the read was not journaled. Its unchanged siblings still leave such a ring unanswered, so the bell's 30-second window also fires. That is a second wake the record never states.
- Evidence:
  - D3's first paragraph, unchanged, says "The ring's own claim marks the ring answered at the acknowledgement". It counts only a journaled acknowledgement, as D3's release sentence shows: "Every way the ring's answer path ends before the daemon journals the acknowledgement releases the claim".
  - The new exit-0 sentence ties the unanswered ring to the bell: "the ring stays unanswered, and the bell's 30-second window for an unanswered ring applies as today".
  - So when the daemon refuses the acknowledgement, or releases the read before the confirmation is lost, the model wakes with the mail and the bell is typed later too. The new Bound ("shows the mail the daemon did not journal twice, once in the wake and once at a later claim") and the D3 hook-wait acceptance bullet leave the bell out.

**LOW: D5 and D3 name different outcomes for a journal failure on the ring path**
- Changed text: D5's new outcome, "`ack-failed` (the client acknowledged and the daemon failed to journal the read, with the error)".
- Summary: D5 gives the ring path two outcomes for one event.
- Evidence:
  - D3, unchanged, releases the claim for every ring end before the daemon journals the acknowledgement.
  - D5's unchanged next sentence says "`released` includes every end of the ring's answer path before the daemon journals the acknowledgement".
  - A failed journal write is one of those ends, so it is logged as both `ack-failed` and `released`. The fix did not say which one the ring logs.

**LOW: D6 keeps only lines, yet D5 logs ids for the echo**
- Changed text: D6, "it keeps the ring's lines, in memory, as the strand's echo", next to D5's "the ids of D6's echo it hands out" and D6's Bound "D5's `ring` line records the ids".
- Summary: D6's echo holds lines alone, but D5 logs the echo's ids.
- Evidence: D6 never says the echo keeps the ring's message ids. D5's claim line and the D5 acceptance bullet ("ids, echo ids and outcome") rely on them.

Further tool calls: none. My context already held every changed and sibling text.
