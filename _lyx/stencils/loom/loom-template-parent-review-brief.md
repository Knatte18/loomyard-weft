<!-- This is the parent-review reviewer brief, handed to the one-shot fork the parent runs for a review notice, rendered by ParentReviewBrief (internal/loomengine/parentreview.go).
     Every marker is a top-level {{.X}} substitution: {{.slug}}, {{.decision_record_path}}, {{.support_log_path}} and {{.review_format_path}}.
     The review-output format is named by the path of the deployed burler-review-format stencil and never restated here.
lyx-stencil: sha256=481e8c53e2c1787bd625b2a2196d7397db3de6e7102e2de5bd041710ea7f2ad5 build=414d8b713a9cb4a85bd49c3a8e9b733cfb38e7b5 time=2026-10-11T19:08:33Z -->

# Parent review — does the design for {{.slug}} fit the planned work?

You are reviewing a child task's discussion on behalf of its parent.
The child has finished its design and is waiting on your verdict.

## What to read

1. The decision record: {{.decision_record_path}}
2. The support log: {{.support_log_path}}
3. The board: run `lyx board list`, then `lyx board get` every entry that touches this task.

## What to judge

This is a scope check only: judge whether the design does what this task's board entry asks, no more and no less, without duplicating or contradicting any other board entry.
A finding is a scope deviation: a goal or Done-when point left out, work that belongs to another entry, or a direction the entry rules out.
Do not judge design details or the design's internal quality; the Discussion-Review segment reviews those in detail right after you.

## Write the review

Write your findings in the format that {{.review_format_path}} defines.
Read that file for the format and follow it; this brief does not restate it.

A review with any finding is a reject.
An approve carries no finding the writer must act on.

## Submit the verdict

Submit with exactly one of these:

- Approve: `lyx loom review approve {{.slug}}`
- Reject: `lyx loom review reject {{.slug}} <review-file>`, where `<review-file>` is the review you wrote.

The verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
