---
round: 1
focus:
  - "Card 2 (final-summary-spec-rename): check the rename used `git mv` first with only surgical edits, and that no stale reference to the old spec path remains in code, docs, stencils or embed/registration lists."
  - "Cards 6 and 7 (`--apply`): check that every mutating path in `lyx fabric unwire`, `lyx board prune` and `lyx board retire-legacy` is gated on `--apply`, and that a bare run prints the applied run's fields plus `\"applied\": false` with nothing written."
  - "Card 4 (batten-terminal-marker-reset): check the marker is cleared only where batten seeds a fresh status and never on a child re-spawn, including the resume and re-spawn paths."
  - "Card 5 (reconcile-floor-migrate): check each of the shuttle, orch and loom `Migrate` hooks raises only a below-floor key, leaves at-or-above values untouched, and that the rewrite is reported."
  - "Card 9 (carry-over-concurrent-write): check `WriteCarryOver` detects a changed record both before the rename (refuses) and after it (errors), and that the check cannot misfire on the writer's own write."
  - "Card 8 (index-note-audience): check an index note prints only under the audience its annotation names, and that notes without an annotation keep their prior behavior."
  - "Test economy and budget: check new top-level tests in cards 5 and 9 each raise their package row in `cmd/lyx/testdata/test-budget.yaml`, and that no other card adds a top-level test that duplicates an existing one."
  - "Docs-in-same-commit: check observable CLI changes (cards 5 to 9) update the affected `doc.go` files and `docs/overview.md` where the project's CLAUDE.md requires it."
---

Focus is derived from the plan's Card Index and Shared Decisions.
Entries name where to look; none pre-states a verdict.
