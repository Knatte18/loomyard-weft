---
round: 1
focus:
  - "Card 6 (granularity): it bundles the engine split, the handoff, the seams, and every caller (burlercli, loomcli, shedadapters, shedrecipe, shedfake) in one card. Settle whether that is one independently reviewable unit or whether a public-surface symbol (for example `ModelChoice`/`RoundModels` it Uses, `StrandRemover`, `Handle`) is hidden inside it and deserves its own card."
  - "Fidelity, decision coverage: trace every Decision D1-D12 and every Constraint in `_lyx/discussion/decision-record.md` to a carrying card. In particular D8 (the `Disputed` section and the prior-rounds wording), D9 (scalar-or-list parse, empty-list error, all-entries resolve at run start, template defaults), D10 (recipe-row key removal and the `loom-recipe.yaml` comments) and the docs-in-same-commit list (`PATTERN.md`, `pattern/PATTERN-review-round.md`, both `doc.go` files, `loom.yaml` template comments)."
  - "Fidelity, scope: settle whether any card edits a file or adds behavior the decision record does not license, including the Notes-for-the-plan-writer line that a worktree's `_lyx/config/loom.yaml` is not this task's to edit."
  - "Test economy: for each card that plans new tests (cards 1, 3, 4, 6, 7), settle whether a planned test duplicates behavior an existing test already covers, or is one test per symbol rather than per public-surface behavior. Name the existing test."
  - "Attack-surface: for each new verb, flag, routing edge or removed or weakened guard (`ProbeGated`, `await-review`, the ready marker, the four `lyx burler run` flags, the resume branch that stops and respawns halves, the `ErrHalfNotStopped` no-retry exit), settle whether the card's `**Intent:**` states what it can let through and what bounds it."
  - "Verify coverage: settle whether every package any card targets has its tests run by `## verify:`, and whether the section follows the order the plan template states."
  - "Card 7 and card 6 sequencing: card 6 keeps the pair-file attach probe until card 7 replaces it. Settle whether the tree builds and tests pass after card 6 alone, and whether the resume rules (D6) are fully carried by card 7 with `Uses` that resolve."
---

Focus entries are questions to settle, not verdicts.
The plan's mechanical checks, derived dependency edges and `ImpactSummary` requirements by card type are left to the gate and are not raised.
