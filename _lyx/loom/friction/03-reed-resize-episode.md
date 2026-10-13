# Card 8: the single-key reason dedupe cannot hold the halted arm's alternating reasons

Card 8 specifies a `childWait.reasonKey` field that `report` compares against the new key, logging at Info only on a difference.
The halted arm reports two reasons per check while a Publish verify retry waits: `publishRetryStep` reports the retry reason, then `haltedStep` reports its own, so the last key flips on every check and every check logs two Info lines.
The card's own test row (a busy-goto refusal before a pause ends the wait) also needs one line for the retry reason, which a single last key cannot give.

I replaced the field with `reasonKeys map[string]bool`, which logs a key once per Call.
A Call ends when the child's state changes, so that is still once per state of the child.
The card should name a set of keys, or say where the halted arm's second report goes.
