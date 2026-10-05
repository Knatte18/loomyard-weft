---
round: 1
focus:
  - "D4 and Acceptance criteria: the seed of the hub `board.yaml` is said to be committed in `_board`, but D4 names no commit step, lock or `Bolt` route for it (only D5 does, for edits). Settle whether the decision record states how the seed reaches git and what bounds it, or whether the plan writer would have to invent it."
  - "D6 and the attack-surface question: `--apply` deletes a per-worktree copy of a hub-wide module once the hub file exists, with no stated check that the copy's content is already in the hub file. Settle whether a pair's unmerged vocabulary edit can be silently dropped, and whether the stated Bound covers that."
  - "D5: settle what the decision says happens when the `_board` commit or push fails after the file was already written (edit on disk, nothing committed), and whether the stated Bound covers any guard the strict load or the write lock used to provide."
  - "Open risks versus Decisions: the seed-source report (`reconcile result reports which source seeded it`) and the post-deploy `lyx fabric reconcile` step live only under Open risks. Settle whether each is a requirement the plan writer needs and so belongs in a Decision or Acceptance criterion (completeness-before-leanness), or is correctly only a risk."
  - "Auto-mode assumptions: the prime-resolution premise (the worktree on the warp default branch, found by worktree listing) carries D4's seed source. Settle whether it is a false or unverified premise, including a hub with no checked-out prime, and whether D4's template fallback is consistent with it."
  - "Relocation: settle whether any `decision-record.md` content is deliberation or plan-level detail (for example the `boardengine/doc.go` paragraph rewrite under Auto-mode assumptions, or Interview-style survey text in D1 rationale) that belongs in `support-log.md` or the plan, after extracting any requirement into Decisions or Constraints first."
---

Focus entries respect the rubric's `Do not flag` list: none asks for a "Notes for the plan writer" subsection, for rejected alternatives in the decision record, or for a complete call-site enumeration.
The stencil tells the writer not to gather exact signatures, `file:line` citations or dependency lists, so no entry asks for them.
