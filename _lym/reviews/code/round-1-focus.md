---
round: 1
focus:
  - "Scope against the board entry's Idea: it names the darn writer, the Webster fork and the fixers. The diff adds the clean gate to Darn-Write and to the Code-Burler-Fix rows of the darn, sampler and tapestry recipes, and re-prompts the Parent-Align and Finalize fixer. Check whether the Webster fork's end gate, and every other producing row that runs a session, is covered or left out without a stated reason."
  - "internal/landingshed/fixerrun.go `judge`: the dirty path now charges a turn and re-prompts while `s.rec.Spent < s.cap`. Check that the cap comparison matches how the spent end is decided for a failing landing gate in `check`, that a charged turn cannot loop past the cap, and that the findings reach the fixer on the next turn."
  - "internal/loomshed/cleangate.go and its resolution in internal/shedrecipe/entries_gate.go: check the `clean` gate is a must-pass gate whose `attempts` cap behaves like the other gates', that a `pass_on_cap` setting on it cannot let a dirty tree through unnoticed, and that the name is reachable in every Env a recipe row can build (WorktreeRoot absolute)."
  - "Gate ordering and interaction: the clean gate is listed first, before verify and description. Check that a file the verify or description gate itself writes or leaves in the worktree (logs, reports, the decision record, `_lym` output) cannot make the clean gate fail or loop, and that git-ignored paths are excluded as claimed."
  - "Test economy and coverage: internal/loomshed/cleangate_integration_test.go and the changed internal/landingshed/fixerrun_test.go. Check that the new behaviors (clean gate pass and fail, fixer re-prompt on a dirty tree, the dirty end after the last turn) are covered, and that no new test duplicates an existing one."
  - "Repository rules and docs: internal/loomshed/doc.go, internal/landingshed/doc.go and internal/verifytree/doc.go were edited. Check the constraints document's rules for package docs and for a recipe contract change land in the same diff, and that each commit message's claims match the diff."
---

Round 1 seeds the focus from the board entry's Goal and the committed diff over the review range.
