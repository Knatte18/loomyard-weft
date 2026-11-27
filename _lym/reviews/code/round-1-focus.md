---
round: 1
focus:
  - "Retry coverage: trace every remote lookup Worktree-Create reaches before it can block, in internal/fabricengine and the batten row that calls Add, including the reported `ls-remote` of the weft branch. Which of them run inside `retryOriginProbeInTransit`, and which return a transport failure with no retry?"
  - "Classification and re-run safety in internal/fabricengine/add.go: do the errors from `remoteHeadTip` and every path of `weftLiveness` reach `gitexec.IsTransportFailure` in a form it recognises, including a failure that is swallowed or folded into the probe result instead of returned? Is each probe safe to run again from the start while the other runs concurrently?"
  - "Stop-question premise in internal/battencli/drive.go: what evidence do the tackler and arbiter get when a run stops at Worktree-Create? Does any other path still hand them text claiming the task worktree was created? Do `taskWorktreePresent` and `taskWorktreeLocation` leave every real failure logged and every absent-pair case quiet?"
  - "Truth of the new prompt text in `battenTacklerRecipe` and `battenArbiterRecipe`: is `never a pair to restore or abandon` correct for every state of a run whose Worktree-Create never finished done, including a partly made pair, a leftover branch from a removed pair, and a failure after Add created something? Check it against the Worktree-Create row and the tackler's and arbiter's allowed moves."
  - "Coverage: which behaviour added in internal/battencli/drive.go has a test, both the skipped usage recorder for an absent pair and the changed prompt text's effect? Does `TestAdd_RetriesOriginProbesThatFailInTransit` exercise both probes as its doc comment states in each of its rows?"
---

Seed for round 1, drawn from the committed range and the board entry under the record's `## Goal`.
The entry asks for two things: a retried remote lookup at Worktree-Create, and a stop question that reads an absent branch on an unmade pair as the fresh state.
