---
round: 3
focus:
  - "Cards for the batten cancel export and the board removal (`_lym/plan/04-batten-cancel-export.md`, `_lym/plan/06-*`): settle whether some named test drives `Cancel`'s conversion of an end-only cancel report into an error through fakes, so a `Cancel` returning nil for an end-only report would fail a named test, and whether `Remove` step 1 then keeps record and entry."
  - "Card `_lym/plan/13-check-fix-loop.md` and the cards editing `runCheck` in `internal/adoptcli/check.go`: settle whether every notice `runCheck` posts, the failed-tier notice included, has a way forward consistent with the fix loop and naming the worktree the gate ran in."
  - "Card `_lym/plan/11-adopt-holds.md`, `reconcileWarnings`: settle whether a branch the reconcile moved (state ahead with `Moved` set) is excluded from the unmoved-branch warning, and whether the worktree named after `lym fabric pull in` is chosen per branch kind."
  - "Cards editing `docs/overview.md` and `internal/battenshed/doc.go`: settle whether every sentence that says `lym adopt` starts a run's first pass is reworded by some card."
---

Targets the areas where round 2 found gaps, so round 3 checks the fixes there and keeps the rest of the plan at the rubric's usual reading.
