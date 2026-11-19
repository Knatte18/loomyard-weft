```yaml
lens: code-scope
outcome: reported
```

The code-scope lens traces every change in the diff to a goal point of the board entry, and every goal point to a change or a recorded reason.

**Decisions.** The record has no `Added after Discussion` entries, so the goal points come from the board entry under `## Goal`:
- **G1 (#151):** on invalid review-file frontmatter, send the parse error back to the reviewer for a repair attempt before failing the run.
- **G2 (#151):** the review prompt states that a frontmatter value holding `:` must be quoted.
- **G3 (#151):** the halt note says which producer's output was invalid, and why the preceding `stuck` step did not stop the run first.
- **G4 (#152):** send a repair prompt that quotes the YAML error and retry the same attempt instead of halting.
- **G5 (#152):** consider a verb that serialises the frontmatter.
- **G6 (#152):** name the failing segment consistently in the halt note's `producer` and `history` lines.

**Per change:**
- **C1, `claudeengine` cut-label top rule** (`session.go` `isTopBoxRule`, `cutLabelRuleEnd` and the `TestInputBoxText` rows): traces to G1 and G4.
  - The repair path already exists: `ReviewGate` puts `err.Error()` into the findings at `internal/burlerengine/verdict.go:251`, and the gate re-prompts with them.
  - The re-prompt was held as "no input box" until the bound ran out (`internal/shuttleengine/wait.go:618-622`).
  - The C1 commit message (`3c5f4fdc`) names that hold as the cause, so the change makes the requested repair turn reach the reviewer.
- **C2, quoting rules in `contracts/stencils/burler/burler-review-format.md:45-52`:** traces to G2.
- **C3, `burlerengine` `quotingHint` and its `TestParseReview` row:** traces to G1 and G4. The hint rides inside the parse error that the gate re-prompt quotes back to the reviewer.
- **C4, `haltnote` `RoutedTo` with `routed to <producer>`, plus `loomcli` `writeHaltNote` passing the route on and the `halt_test` update:** traces to G3 and G6.
  - `contracts/recipes/darn-recipe.yaml:160-163` shows `Code-Bouncer` has `on_stuck: Code-Burler-Review`.
  - So the history row now reads `Code-Bouncer / stuck / … / routed to Code-Burler-Review`, matching the `producer: Code-Burler-Review` line.
  - That explains why the `stuck` did not stop the run.

**Per decision:**
- G1 and G4: C1 and C3, on the existing three-attempt gate repair path.
- G2: C2.
- G3 and G6: C4.
- G5: no change. The goal point only asks to consider the verb, and `_lym/landing/summary.md:46-47` records it was considered and left out, with a reason. That settles a "consider" point, so it is no scope gap.

**Findings:** none. Every change traces to a goal point, the only untraced edits are tests forced by traced changes, and no goal point is left without a change or a recorded reason.

Tool calls:
1. Grepped the `Code-Bouncer` name and recipe `on_stuck` lines, to learn where Code-Bouncer's Stuck routes; the manifest did not hold the recipe.
2. Read the `Code-Bouncer` row in `contracts/recipes/darn-recipe.yaml`, to confirm `on_stuck: Code-Burler-Review` for the G3/G6 trace; the recipe row was not in context.
