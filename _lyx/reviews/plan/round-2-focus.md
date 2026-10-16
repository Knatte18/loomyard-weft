---
round: 2
focus:
  - "Card 2 (`_lyx/plan/02-verify-package-missing.md`): does every guard branch of its `missingPackageArgs` chain walk (a `cd` in a pipeline stage, a `cd` after `||`, a `-C` that cannot be placed, an unplaceable `cd`) have a named test input in `TestValidate_VerifyPackageMissing` that reaches it, given the walk is a separate loop from `crossModuleCommands` in `internal/planparser/verifymodule.go`?"
  - "Card 2: does its Intent license every case where a package argument passes unreported against the answer key's D3 bound, in particular any argument resolving outside the worktree root such as `../sibling`?"
  - "Card 4 (`_lyx/plan/04-loop-status-stop.md`): for a status read error, is the outcome stated and tested in every state of the loop's single re-step, including when an earlier transient child failure has already used it?"
---
