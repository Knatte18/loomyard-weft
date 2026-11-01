---
round: 2
focus:
  - "Card 4 exports `plan:internal/battencli#Cancel` and card 6's `fronttask.Remove` step 1 consumes it. In `internal/battencli/cancel.go`, `judgeOrigin` sets `endOnly` with a nil error when origin is unreachable, and `cancelRun` returns before `clearRunPhase`. Settle whether the plan makes `Remove` fail the cancel step, keep the record and the board entry, and name `lym batten cancel <slug> --remove` first in `RemoveError.HandRemoval` for that outcome, as the decision record's cancel-refusal rule requires, and whether a named test row reaches it."
  - "Card 11's `reconcileWarnings` warning and its `TestAdopt` rows: settle whether every warning line names `lym fabric pull` in that worktree as the decision record's D2 requires, including for a diverged code branch whose reconcile outcome carries its own way forward."
  - "Card 13's `failedEnd` filing step: settle whether the label set on the fix task is accepted by `validateLabels` in `internal/boardengine/labels.go` on a hub whose `board.yaml` lacks the `gate` label, and whether the plan or a recorded decision settles the choice."
  - "Cards 6 and 13: settle whether an own fix task whose entry holds a run status with no readable batten run record is classified by `failedEnd`, and whether `Settle`'s drop of records for a project the machine no longer owns has a stated bound on a repeated adopt check."
  - "Cards 5, 11 and 12: settle whether the doc and header texts that still say `lym adopt` resumes runs or runs the check in the prime are covered by some card's Edit list or Intent, and whether any proposed target addition passes `redundant-file-target` and `redundant-package-target`."
---

Focus targets the open round-1 defects without restating their findings.
