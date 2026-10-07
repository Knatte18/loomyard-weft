---
round: 4
focus:
  - "Settle in decision-record.md D4 how the `publish_verify` value reaches the parent branch: landing config is per-worktree (`landing` is not `HubWide` in `configreg`) and `internal/landingshed/finalize.go` (`configChangeNotice`) tells the hub to re-apply a change on the parent's copy rather than carrying it. Question: does D4 state the true condition under which a Publish runs the key, the rollout step, and what a task forked before the re-apply skips?"
  - "Settle in D6 and D7 whether the lint's checked set is defined consistently: D7 checks a break when its line or the next line is added or changed, and D6 says a fix rewraps only the touched sentence and never flags a break between two untouched lines. Question: on a legacy fixed-column block, does fixing one flagged break make the next break checked, and does the record define the checked set so that it cannot?"
  - "Settle in D7 which rounds have no base and which commits no gate lints: D2's fallbacks include a missing recorded plan-verify pass and a non-ancestor commit (no base) and also `go.mod` change, unmappable file, `go list` failure and a too-long command (base exists). Question: does D7 name exactly the no-base cases, and does it bound the fixer commits made after a card gate and before the first round's base?"
  - "Check that D2 and D5 bounds match the recipe in `contracts/recipes/loom-recipe.yaml`: Publish and Finalize carry no `on_stuck`, and the plan verify does not run the `tmux` or `llm` tier. Question: do the stated costs and coverage claims in D2, D4 and D5 hold for a landing into a parent outside `require_pr_to_base`?"
---

Gating keys to settle are the three open design keys; the fourth entry covers the consistency claims that depend on them.
