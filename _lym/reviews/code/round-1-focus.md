---
round: 1
focus:
  - "`contracts/stencils/engine/claude/edit-directive.md` line 9: the Goal asks to delete only the `cat`, `head` and `sed -n` reading clause. Check the rewrite keeps every other rule of the sentence (override of harness text for `sed`, heredocs and short scripts) and the Read-with-offset rules on the next lines."
  - "The Goal names a second file, `contracts/stencils/edit/edit-directive.md`, which does not exist in the tree. Check that no other stencil, spec or doc still carries the deleted clause, and that the Codex directive's reading line is outside the Goal's scope."
  - "`internal/editdirective/doc.go`: check the package comment's new sentence matches the stencil as the diff leaves it, and that nothing in the package (code or tests) still depends on the removed reading wording."
  - "`contracts/stencils/loom/loom-rubric-discussion-review.md`: check the new `An unpinned tool fact` bullet is a rubric line the Goal asks for, sits in the right list, and does not contradict or duplicate its neighbouring bullets (the research-question and board-entry bullets)."
  - "Check the new rubric bullet's list of pins (test, recorded command and output, documentation) and its trigger (Decision or Constraint) against the Goal's idea: a decision resting on a tool's behavior states the observed result and where it is pinned."
  - "Check the commit messages and that the diff makes no change beyond the two retro proposals."
---

Both changes are prompt text plus one package comment.
No Go behavior changed and no tests were added, so test economy is a quick check of the diff's absence of new tests.
