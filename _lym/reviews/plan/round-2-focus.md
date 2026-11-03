---
round: 2
focus:
  - "Card 2 (`_lym/plan/02-record-follows-entry.md`): settle whether every error a saved `Board.RemoveTask`, `Board.Prune`, `Board.MergeTasks` or rename can now return after the save is one D4 prescribes, and whether the Board methods, board verbs and `plugins/ly/skills/board/SKILL.md` that callers read as 'nothing saved' document it."
  - "Card 2 rename path: settle whether a flagged entry renamed onto a slug whose `decision-<new>.md` already exists can still serve that file as its record, and whether a named test row reaches that case."
  - "Card 7 (`_lym/plan/07-discussion-approve-seam.md`): settle whether the store-failure handling in `approveDiscussion` (done entry, absent entry, unopenable board) matches D6's rule that a failed store fails the settle, or is recorded as a departure, and whether `TestApproveDiscussion_Real` rows and the comments in `tapestry-recipe.yaml` and `TestApproveSeam_FailsToBuild` agree with the final rule."
  - "Cards 8 and 9: for each error branch around `PastDiscussionApproval` and `committed()` in `copyRecordToBoard`, `wayCommit` and `approvalRow`, settle whether an input and a test reach it past the upstream recipe-parse and commit checks in `internal/loomcli/decision.go`."
  - "Card 2's refused-rename test row and card 5's `internal/boardcli/doc.go` edit: settle whether the row builds a board with a refusing `ValidateSlug`, and whether the doc's stored-value-only payload sentence names every key the plan makes stored-only."
---

Focus targets the open design-class finding first, then the decision-class departure and the unreachable-branch and doc consistency items.
