# Parent review — does the design for test-tiers fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/test-tiers/_lyx/discussion/decision-record.md
2. The support log: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/test-tiers/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/test-tiers/_lyx/shed/test-tiers/alignment/parent-review/attempt-1/report-f682ff60c365209d.md
Why it did not settle the review: Board entry test-tiers says zero discovered tests is a failure; decision D7 in the test-tiers decision record keeps that only for whole unfiltered runs (as a min_tests floor) and lets a narrower run (computed, --base, --card or explicit scope, or words after --) that discovers zero tests pass with a warning, since such a scope can legitimately hold no tests. Should a narrower zero-test run fail as the board says, or pass with a warning as D7 says?

When the report is not `none`, read it first; it covers the board and the other runs in flight.
Both read `none` when no agent ran.

## What to judge

This is a scope check only: judge whether the design does what this task's board entry asks, no more and no less, without duplicating or contradicting any other board entry.
A finding is a scope deviation: a goal or Done-when point left out, work that belongs to another entry, or a direction the entry rules out.
Do not judge design details or the design's internal quality; the Discussion-Review segment reviews those in detail right after you.

## Write the review

Write your findings in the format that /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/_board/_lyx/stencils/burler/burler-review-format.md defines.
Read that file for the format and follow it; this brief does not restate it.

A review with any finding is a reject.
An approve carries no finding the writer must act on.

## Submit the verdict

Submit with exactly one of these:

- Approve: `lyx loom review approve test-tiers`
- Reject: `lyx loom review reject test-tiers <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
