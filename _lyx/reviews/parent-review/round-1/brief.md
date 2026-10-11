# Parent review — does the design for driver-arbiter fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/driver-arbiter/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/driver-arbiter/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

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

- Approve: `lyx loom review approve driver-arbiter`
- Reject: `lyx loom review reject driver-arbiter <review-file>`, where `<review-file>` is the review you wrote.
