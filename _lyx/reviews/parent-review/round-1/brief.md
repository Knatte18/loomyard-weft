# Parent review — does the design for pause-esc-rewind fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: /home/knatte/Code/loomyard-LYXHUB/pause-esc-rewind/_lyx/discussion/decision-record.md
2. The support log: /home/knatte/Code/loomyard-LYXHUB/pause-esc-rewind/_lyx/discussion/support-log.md
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## The alignment agent

An alignment agent may have looked at this task before you.
Its report: /home/knatte/Code/loomyard-LYXHUB/pause-esc-rewind/_lyx/shed/pause-esc-rewind/alignment/parent-review/attempt-1/report-e3aa9ce22409ad24.md
Why it did not settle the review: Overlap with reed-multi-window (Discussion-Burler-Review, no plan yet): both runs plan changes to internal/shuttleengine and internal/shuttleengine/claudeengine. reed-multi-window changes spec.go and attach.go (own-window anchor default), adds lyx shuttle open, and adds a launch line, session name flag and turnless resume in claudeengine. pause-esc-rewind changes Runner.Interrupt and Run.Interrupt, extracts a running-turn predicate from readiness.go and sessionstate.go, and adds a running-turn pane reading and a settle time in claudeengine. The named files differ, so a text conflict is unlikely. Which lands first, or may both proceed in parallel with the later one rebasing over the earlier in these two packages?

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

- Approve: `lyx loom review approve pause-esc-rewind`
- Reject: `lyx loom review reject pause-esc-rewind <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
