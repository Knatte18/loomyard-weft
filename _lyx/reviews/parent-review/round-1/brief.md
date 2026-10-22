# Parent review — does the design for orch-strand-cycle fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/orch-strand-cycle/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/orch-strand-cycle/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/orch-strand-cycle/_lyx/shed/orch-strand-cycle/alignment/parent-review/attempt-1/report-4c4db781c2e30c26.md
Why it did not settle the review: The board entry orch-strand-cycle (operator decision 2026-10-21) says a strand is live only when its pane sits in the reed session and window it was launched into and its foreground process is the engine (claude), not a shell. Decision record D1 departs three ways: (1) it checks the reed session only, not the window, so a pane moved to another window of the session still reads live and start launches no second orch beside it; (2) it tests that the foreground command is not reed's launch shell rather than that it is claude, keeping reed engine-neutral per PATTERN-shuttle-provider-seam, so another program left running after the engine exited reads live; (3) it skips the foreground check on Windows, so there an exited engine reads live as today. Accept each departure, or hold the run to the entry's rule for it?

When the report is not `none`, read it first; it covers the board and the other runs in flight.
Both read `none` when no agent ran.

## What to judge

This is a scope check only: judge whether the design does what this task's board entry asks, no more and no less, without duplicating or contradicting any other board entry.
A finding is a scope deviation: a goal or Done-when point left out, work that belongs to another entry, or a direction the entry rules out.
Do not judge design details or the design's internal quality; the Discussion-Review segment reviews those in detail right after you.

## Write the review

Write your findings in the format that /home/knatte/Code/loomyard-LYXHUB/_board/_lyx/stencils/burler/burler-review-format.md defines.
Read that file for the format and follow it; this brief does not restate it.

A review with any finding is a reject.
An approve carries no finding the writer must act on.

## Submit the verdict

Submit with exactly one of these:

- Approve: `lyx loom review approve orch-strand-cycle`
- Reject: `lyx loom review reject orch-strand-cycle <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
