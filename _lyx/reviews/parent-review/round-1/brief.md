# Parent review — does the design for go-driver-assistant fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/go-driver-assistant/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/go-driver-assistant/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/go-driver-assistant/_lyx/shed/go-driver-assistant/alignment/parent-review/attempt-1/report-7157d1955720e6cd.md
Why it did not settle the review: 1. orch-split records the operator's decision of 2026-10-12 that a run's driver receives every instruction for the run. This run's D15 (auto-pick Q11) instead has the orch address a go-driven run only through lyx loom and lyx shed verbs, with no receiver, until a later task adds one. Do you accept that departure? 2. judge-cap-call (Discussion-Write) and this run both change contracts/stencils/loom/loom-template-arbiter.md. There judge-cap-call D7 rewrites the review rule, and this run's D14 adds a case-brief marker and the check-the-facts text. Both also add loom config keys in internal/loomengine and rewrite the Arbiter section of internal/loomcli/doc.go. Which run lands first, so the other rebases onto it?

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

- Approve: `lyx loom review approve go-driver-assistant`
- Reject: `lyx loom review reject go-driver-assistant <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
