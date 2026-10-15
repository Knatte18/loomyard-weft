# Parent review — does the design for kickstart-per-card fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/kickstart-per-card/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/kickstart-per-card/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/kickstart-per-card/_lyx/shed/kickstart-per-card/alignment/parent-review/attempt-1/report-872482bf91c91098.md
Why it did not settle the review: Peer glyph-audit-tool's decision record requires a task that adds a pack layout to add its marker row to tools/forkaudit in the same task. kickstart-per-card's decision record (Scope Out, Open risks) leaves that to the orch after landing. Its per-card prompt also carries the "## Start here" heading that forkaudit's marker table reads as the file arm. Should kickstart-per-card add a per-card marker row, ordered ahead of file (for example matching "lyx loom card-pack"), and a span source to tools/forkaudit at Merge-In? Or should glyph-audit-tool drop that constraint and change its file-arm marker?

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

- Approve: `lyx loom review approve kickstart-per-card`
- Reject: `lyx loom review reject kickstart-per-card <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
