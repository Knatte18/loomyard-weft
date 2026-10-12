# Parent review — does the design for kickstart-read-all-first fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/kickstart-read-all-first/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/kickstart-read-all-first/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/kickstart-read-all-first/_lyx/shed/kickstart-read-all-first/alignment/parent-review/attempt-1/report-b5066266e0cb5ce9.md
Why it did not settle the review: Peer run-robustness (running, Webster) changes internal/websterengine: verifygate.go for its D5 and D6 (card 12, webster-hang-gate) and the package doc.go to describe them, while this run edits internal/websterengine/doc.go, template_test.go and beginbatch_test.go. Which run lands first, or how is the doc.go edit split between them? Peer lyx-halt's decision record also lists contracts/stencils/claims_test.go and webster package docs among its edits, which this run also changes; its branch has not touched them yet.

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

- Approve: `lyx loom review approve kickstart-read-all-first`
- Reject: `lyx loom review reject kickstart-read-all-first <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
