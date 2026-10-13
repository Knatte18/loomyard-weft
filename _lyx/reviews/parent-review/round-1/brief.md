# Parent review — does the design for arbiter-fixes fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/arbiter-fixes/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/arbiter-fixes/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/arbiter-fixes/_lyx/shed/arbiter-fixes/alignment/parent-review/attempt-1/report-2cfca6a9ea103aaa.md
Why it did not settle the review: Peer overlaps to settle. (1) logging-cleanup (Plan-Bouncer) and arbiter-fixes both change contracts/stencils/claims_test.go: logging-cleanup the orch stencil's rows, arbiter-fixes the implementer row that pins deviations: (line 451) and likely the alignment and arbiter rows. By the landing-order rule logging-cleanup lands first, and arbiter-fixes takes its changes at Merge-In. (2) twb-ftb-1 (Discussion-Write, bench at base 800a0e892) and arbiter-fixes both change internal/websterengine (twb: Master's shell wait, verify re-prompt, recovery evidence; here: record-batch's scope warnings, the digest, digestSummaryLine, ParseReport), internal/planglyph (twb: a build-variant predicate in DoneChecks; here: ScopeGuard's file field and file-level comparison), webster-template-master.md, refusal-spec.md, and the websterengine and planglyph doc.go files. twb-fta-1 has the same board entry and no record yet. By the rule arbiter-fixes lands first; will the twb bench runs land at all? If not, they need no landing order. Either way, may arbiter-fixes go on with these text-only overlaps?

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

- Approve: `lyx loom review approve arbiter-fixes`
- Reject: `lyx loom review reject arbiter-fixes <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
