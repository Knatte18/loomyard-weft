---
round: 2
focus:
  - "Refusal-spec rows: for every card in `_lyx/plan/` that adds, rewords or removes a refusal, check that its `**Edit:**` list names `contracts/specs/refusal-spec.md` and names each existing row it rewrites or deletes, including Plan-Write's rotation in the stale-plan card (webster busy, archive collision) and the reset, config-invalid and rebaseline cards. The question is whether a stale or overlapping row survives."
  - "Test economy in the begin-batch, reset and reconcile-migration cards: for each planned test, name the existing test (`TestBeginBatch_Refusals`, `TestPlanReset`, `TestResetCmd`, the config migration table test) that already covers the behavior at the same surface, and check that the plan extends it instead of adding a new function, and that `PlanReset` (read-only) is not asked to assert a HEAD move."
  - "Build order across cards that rename a shared field or change an exported signature (the batcher orientation rename and the `PlanEditError` / `fingerprintMismatchWayForward` re-entry parameter): grep every caller, including `tools/tokencount` and `webstercli`, and check that each caller is edited in the same card or an earlier one."
  - "Standalone reset: check that the reset-to-start card states whether `PlanReset`'s foreign-dirty-path refusal runs in standalone mode, and that the per-mode tree outcome, the keep-refusal way forward and the matching tests follow that single choice."
---

Entries restate their sites and name where to look.
