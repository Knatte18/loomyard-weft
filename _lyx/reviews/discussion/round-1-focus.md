---
round: 1
focus:
  - "decision-record.md Decisions (once-per-run marker) against Open risks (run resumed from an older build) and Acceptance criteria: settle whether the gate is defined as 'no once-marker present' or 'first spawn of the run', since the resumed-run risk only holds under the second reading while the marker decision implies the first, and innerrun.go clears the sibling spawn-confirmed marker before every spawn attempt."
  - "decision-record.md Decisions and Constraints on the once-marker living in Run-Shed's ephemeral scratch directory: settle what happens to the once-per-run guarantee when scratch is lost between Calls (restart, machine change) and a re-spawn follows, and whether the record states the intended behaviour for that case."
  - "decision-record.md Decisions on the driven path's git hygiene (.git/info/exclude, 'or left alone when git already ignores it') against the Open risk that the exclude file is repo-wide: settle whether the Decisions wording ('the task worktree's .git/info/exclude') and the acceptance criterion ('git status --porcelain is empty ... and .vscode/ is ignored') agree with that shared-file reality and with concurrent batten runs writing the same file."
  - "decision-record.md Decisions on the strand-is-up premise and the attach-only chain: settle whether the record's two load-bearing premises — that Run-Shed's Spawn returns only once the child's driver strand is up, and that the lyx reed attach run on folderOpen in the task folder attaches to the child's own session rather than a fresh one — are stated as requirements the plan writer can act on, given Plan-Write never reads support-log.md."
  - "decision-record.md Acceptance criteria as a whole: settle whether each criterion is observable and testable as written (e.g. 'before the post-spawn status read decides the row's outcome', 'one warning log line naming the slug and the error') and whether any Decision lacks a matching criterion or any criterion lacks a Decision."
  - "Placement across the pair: settle whether any deliberation narrative in decision-record.md (Rationale lines that argue against unchosen options, the illustrative closure signature in Decisions) belongs in support-log.md or nowhere, and conversely whether any requirement currently only in support-log.md (Interview item 6's interactive-workflow reasoning, item 7's mill-block .gitignore finding, the Question ledger's board-get exit-status note) is something the plan writer needs in decision-record.md — applying the completeness-before-leanness test before any move."
---

Seed focus for round 1.
The entries target internal consistency between the once-marker decision, its resume risk and its acceptance criteria;
the git-exclude decision against its own stated repo-wide risk;
whether the two premises the design rests on reach the plan writer;
testability of the acceptance criteria;
and placement between the two files in both directions.

Checked against the rubric's Do-not-flag list and the writer stencil: no entry asks for the optional "Notes for the plan writer" subsection, for rejected alternatives in decision-record.md, for call-site enumeration, or for exact signatures, `file:line` citations or dependency lists the writer was told not to gather.
