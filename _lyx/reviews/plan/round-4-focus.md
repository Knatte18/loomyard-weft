---
round: 4
focus:
  - "Mutating fabric verbs that push through `fabricengine.Bolt.Push`, in the plan cards that rewire pushes to `PushRecorded` or keep `Bolt.Push` as is: for every Go handler that builds a `fabricengine.NewMutations` record and then calls `Bolt.Push`, does a card put that handler on its `Edit:` list and pass its record to the push, so a seed drop appears in that verb's envelope? Settle it by grepping `internal/fabriccli` and `internal/fabricengine` for `NewMutations` and `.Push(` and comparing against each card's `Edit:` list and its stated list of recordless callers."
  - "Callers of `PullThenCommitWritten` that have no envelope, in the pre-run seed pull-first card and the `hubreconcile` pull-first card: does each card state which `*Mutations` it passes and whether a drop recorded in it is logged, consistently with the rule the `Bolt.Push` card states for recordless push callers?"
---

Both entries target the cross-caller completeness of the drop-recording rule: every path that can reset `_board` either carries the drop in its envelope or logs it.
