```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references they touch (fix-consistency lens).

**1. MEDIUM: the D8 accept-refusal bullet contradicts D8's accept and BLOCKING-decision cases**
- **Changed text:** D8, the bullet starting "The escalation runs ahead of the refusal for routing only…", which re-judges the round instead of passing it.
- **Problem:** take a CONVERGED round whose review holds a BLOCKING `decision` finding.
  - `judgmentRefusal` refuses a CONVERGED verdict over any BLOCKING review finding, and re-judging leaves the review untouched.
  - So every `accept` retires the escalation, re-judges, and escalates `decision` again.
  - The round can never settle on accept unless the judge flips to CONTINUE.
- **What it contradicts:**
  - The accept criterion: "`accept` settles, carries every `decision` finding … and goes to Plan-Write."
  - The criterion that escalates "over a BLOCKING `decision` finding without a re-judge", which now loops the next time the parent accepts.
  - D8's new bound "an `accept` passes the round only over a judgment the refusal admits", which no such round can reach.
- **Evidence:** `internal/shedadapters/bouncer.go` `judgmentRefusal` returns "review finding %q is BLOCKING" for any BLOCKING finding in the review. `rejudge` archives only `judgeOutputs`, not the review.

**2. LOW: D10's "can skip or let through" line is now stale**
- **Changed text:** D10, the "With no parent…" bullet, which now "works the entry as written" instead of stopping.
- **Problem:** the line beside it still says "What it can skip or let through: nothing; it only adds a wait." The no-parent path now lets a run land on an entry the writer itself judged plainly outdated, and adds no wait.

**3. LOW: Auto-mode assumptions still name one switch**
- **Changed text:** D8's new fix-row key, and the Constraints line, which now reads "D8's two switches".
- **Problem:** "Auto-mode assumptions" still reads "D8: a Discussion-Bouncer switch, any severity, replacing the fix row's notice there".

**4. LOW: support-log Q11 left stale**
- **Changed text:** D10's first bullet (the parent writes a note *and replies*, and the writer re-reads on any message) and the no-parent bullet.
- **Problem:** `support-log.md` Q11 still reads "ask by `SendMessage` for a decision note, wait held". The fix updated its twin, Q8, but not Q11.

No further tool calls made.
