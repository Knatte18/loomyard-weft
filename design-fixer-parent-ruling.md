# Burler fix round: decision-class rule conflicts with parent ruling mail, fixer has no outlet to parent

`fixer-parent-ruling` · Task · bug, burler, stencil · running · Drift-Expect

Imported from [issue #84](https://github.com/Knatte18/loomyard/issues/84).

What happened:
During a burler plan fix round, the fixer instructions say a `decision`-class finding goes to the parent, and the fixer neither fixes it nor reports on it.
Midway through the round, LOOM mail delivered the parent's ruling on that finding and said to act on it.
The two instructions conflicted.
The fixer followed the mail as the later and more specific instruction and added cards for the ruling.
The fixer-report had no section for such work, so it was recorded in a section of its own, outside the dispositions.

The parent's ruling also asked for something that already exists on HEAD.
The fixer cannot message the parent about the review, so the only outlet for that conflict was a deferral in the fixer-report.

Why it is abnormal:
- The fix instructions do not say what a fixer does when a parent ruling on a `decision`-class finding arrives mid-round.
- The fixer-report format has no place for work done on a ruling, so the work sits outside the dispositions.
- A fixer that finds the parent's ruling mistaken or redundant has no verb to tell the parent, so the conflict surfaces only as a deferral.

What to do:
- Extend the fix instructions to cover a parent ruling on a `decision`-class finding that arrives during the round, and to say it overrides the "neither fix nor report" rule.
- Give the fixer-report a defined section for work done on a ruling.
- Give the fixer a way to flag a mistaken or redundant ruling to the parent, or document that a deferral is the intended channel.

Provenance: task slug `tests-simulator-seam`; note file `burler-plan-r1.md`.
