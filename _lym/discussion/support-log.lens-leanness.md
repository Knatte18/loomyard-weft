```yaml
lens: leanness
outcome: reported
```

Task: the leanness lens review of the decision record and support log.

1. **MEDIUM — Notes for the plan writer, third bullet (D3).** The bullet "check that the start path reaches it" leaves to Plan a fact that D3 depends on, which the rubric counts as a deferred research question.
   Evidence: `grep` finds no `armAt` or `resolveRecipe` call in `internal/loomcli/start.go`, so it is unconfirmed that the start path resolves the recipe before `ensureStatusStrand` (start.go:553).
   `resolveRecipe` also returns `shedrun.RecipeTapestry` when the run has no seed yet. If the first `lym loom start` ensures the status strand before seeding, a pawl run's window would read `tapestry`.
   Fix: settle this from the code now and state the answer in D3. Then cut the bullet.

2. **LOW — Constraints.** Cut the `PATTERN-pane-text-injection` / `PATTERN-wait-mark-display-only` line. Nothing in the design types into a pane or reads a wait mark, so the line is padding.

3. **LOW — Constraints, first bullet.** "reed stays generic…" repeats D1's rationale ("a generic key keeps reed ignorant of beams, loom and pawl"). Keep one: the Constraint, or the rationale clause.

4. **LOW — Scope "In".** The first four bullets restate D1 to D4 almost word for word. Keep only the docs/comments bullet, or point to the Decisions.

5. **LOW — Scope "Out", first bullet, and Auto-mode assumptions, third bullet.** The decision to leave out the pawl row line is stated twice, and its reason is deliberation. Keep one short line in Scope Out; the reason already sits in the support log's Rejected alternatives.

6. **LOW — Open risks vs Auto-mode assumptions, second bullet.** The task window's rename from `overview` to the recipe name is recorded in both places. It is the writer's own choice, not a question only the parent can settle, so the Open risks line is a duplicate. Drop it, or keep it only if the rename is meant to reach the parent for a decision.

7. **LOW — Notes for the plan writer, first bullet.** It lists the files that hold singular-overview text. `relevant.md` already lists those files, and Scope In already names "docs and comments". Cut it, or shorten it to the one thing `relevant.md` lacks (the "Overview anchor" paragraph name).

8. **LOW — D1, the "Everything else about an overview window stays…" paragraph.** This one stays: it guards behaviour the plan must not break. It would sit better as a Constraint than inside the decision.

Tool calls:
- grep of `internal/loomcli/start.go` and `arm.go` for `armAt` / `resolveRecipe` / `ensureStatusStrand`: the exploration never confirmed whether loom start resolves the recipe before the status strand step.
