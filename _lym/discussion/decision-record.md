## Goal

```markdown
## Why (operator, 2026-11-08)

`docs/research/webster/webster-time-2026-11-08.md` found that 8 of 14 Webster runs failed the plan-level verify on its first attempt, with `cmd/lym` among the failing ids in 5 of those 8.
Each such run paid a verify-fix fork plus a second verify, about 6.8 min per affected run and 3.9 min per run on average.
A batch gate tests only the packages its cards touch, so a change that breaks a `cmd/lym` test is first seen at the plan-level verify, after the fork that made it has ended.

## What

- Make the Webster batch gate also run the `cmd/lym` tests (the verify tiers the plan-level verify runs for `cmd/lym`), so the batch's own fork sees and fixes such a failure at once.
- Read how the batch gate's scope is computed in `internal/websterengine` and the gate, and measure from the hub's gate run records (`.lym/gate/runs/*.json`) what a `cmd/lym` run costs.
  From that, choose between adding it to every batch gate and only to the last batch's gate, whichever saves more over the 14 runs in the report, and state the choice and its numbers in the change description.
- Keep this general, not loomyard-specific: a hub names the extra package set through the gate or language config default for Go hubs, with no hub step needed for loomyard (the loomyard default holds `cmd/lym`).
- The plan-level verify stays as it is.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-08): Last batch gate adds the module's main packages, derived, no hub config

Decision: Last batch only, as measured. Under the go scoper the last batch's gate adds every package main of the module (found from the module, e.g. go list), so loomyard gets ./cmd/lym with no hub step and the template carries no Go path. No gates.batch.last_packages field and no lym config gate --set step. Other scopers add nothing. The plan-level verify is untouched.

Rationale: Hub rule: a fix must work without a hub filling in config. The go scoper is already Go-specific, so deriving main packages there hard-codes no language in templates; cmd packages are where cross-package breaks surfaced (5 of 8 verify failures).

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - beginbatch-last-batch-flag-unpinned (class scope, severity MEDIUM, open in rounds 1)
  - detached-runner-main-packages-words-unpinned (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
