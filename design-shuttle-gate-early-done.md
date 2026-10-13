# Shuttle gate ends a twill writer mid-turn

`shuttle-gate-early-done` · Task · bug, shuttle, loom · running · Plan-Burler-Fix

## Seen (twill Plan fork-timing bench, 2026-10-13)

On today's binary (`e17b29219`), two bench runs of twill Plan-Write ended before the writer weighed its lens forks' findings.
In both, the writer wrote the plan, `validate-plan` passed, the ready marker was written and the four lens forks ran.
Right after the last fork finished, shuttle ended the run while the writer was mid-turn:
the trace log shows "turn started after the boundary, holding the gate send" and, about 10 s later, "send released an unmatched turn start held against an idle pane".
The pane looked idle while the model was thinking, so the gate ran and returned Done.
No `lens-review.md` was written and no finding was fixed.

Twill Plan-Write in production can end the same way: the gate does not check that `lens-review.md` exists.
Not yet checked on a real run.

Evidence (transcripts, traces, plans) is in the hub prime's `.scratch/twill-fork-timing-plan/`; the bench pairs `twb-ftp-s-1` and `twb-ftp-r-1` are still standing.

## Scope to discuss

- Why the held turn start is released against a pane that only looks idle, and whether the idle test can see a thinking model.
- Whether the twill gate should refuse Done until the lens review is written.
- Whether real runs since `e17b29219` lost their lens review this way.
