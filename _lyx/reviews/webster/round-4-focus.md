---
round: 4
focus:
  - "Comment line breaks: for every `.go` file the review range changes, walk each comment block, split it into sentences at lines ending in `.`, `?`, `!` or `:`, and list every multi-line sentence that contains a changed line and breaks anywhere other than at a semicolon or a comma before a coordinating conjunction that opens an independent clause, or holds two sentences on one line. `PATTERN-comment-line-breaks` requires one sentence per line, and a changed line makes its whole enclosing sentence a changed comment. Settle whether the whole diff, not only the changed lines, now passes."
  - "Perishable tallies in changed comments: in `internal/fabricengine/destroy.go` and `internal/gitrepo/push.go`, and in any other comment or doc the range changes, does a count of executors, calls, rows or files remain that a later addition would falsify? Settle whether each is derived or worded without the number."
  - "Rubric check-ID coverage: in `contracts/stencils/loom/loom-rubric-plan-review.md`, does every statement of which planparser checks the gate enforces name the set by its exceptions rather than bounding it at a last-row check ID, and do the statements agree with each other?"
  - "Refusal spec coverage: in `contracts/specs/refusal-spec.md`, for the Publish push-rejected refusal, does the row condition cover every outcome of `pushRejectedReason` in `internal/webstercli` or wherever it lives, including the case where `Deps.RemoteOnlyCommits` itself errors and the reason gives the merge way forward?"
---

The first entry is the method check: detect sites mechanically, not by reading.
