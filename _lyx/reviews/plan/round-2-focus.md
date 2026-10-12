---
round: 2
focus:
  - "Card 2 (`_lyx/plan/02-verify-package-missing.md`): its `missingPackageArgs` walks a `## verify:` chain as its own loop, copied from `crossModuleCommands` in `internal/planparser/verifymodule.go`. Settle whether every scan-end branch (a `cd` in a pipeline stage, a `cd` after `||`, a `go -C` that cannot be placed, an unplaceable `cd`) has a named row in `TestValidate_VerifyPackageMissing` that would report a package if the scan went on."
  - "Card 2: its Intent for `missingPackageArgs` must pass only the argument kinds decision D3 in `_lyx/discussion/decision-record.md` lists as passing. Settle whether any pass the card states, such as an argument resolving outside the worktree root, is licensed by D3 and covered by the card's Bound paragraph."
  - "Card 4 (`_lyx/plan/04-loop-status-stop.md`): `runLoop` in `internal/shedverbs/loop.go` keeps one `restepped` flag. Settle whether the card states, and a named subtest reaches, a status read error after the loop's one re-step is already used."
---

Focus entries target the places round 1 touched, restated without reference to the earlier review.
