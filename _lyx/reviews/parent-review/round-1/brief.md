# Parent review — does the design for run-window-bugs fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/run-window-bugs/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/run-window-bugs/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/run-window-bugs/_lyx/shed/run-window-bugs/alignment/parent-review/attempt-1/report-fcf62331a39bbef5.md
Why it did not settle the review: Overlap with judge-cap-call and go-driver-assistant. This run's D6 (lyx loom fmt) changes Go package internal/loomcli, adding a new verb file, a cli.go registration and a doc.go section. Both peers change internal/loomcli, and doc.go is the one file all three edit. This run's D2 and D6 also change Go package contracts/stencils (a comment in roleskills_test.go, and edit/edit-directive.md, which stencils.go embeds), where both peers change stencils.go and claims_test.go. Neither peer's record contradicts a decision here. May run-window-bugs go on beside them, landing after both and taking their loomcli/doc.go changes at Merge-In?

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

- Approve: `lyx loom review approve run-window-bugs`
- Reject: `lyx loom review reject run-window-bugs <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
