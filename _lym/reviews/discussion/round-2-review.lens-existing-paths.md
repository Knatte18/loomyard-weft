```yaml
lens: existing-paths
outcome: reported
```

The task: run the existing-paths lens over the decision record's D1–D11 and trace the code paths that already run against each change.

**Paths traced, per decision**

- D1/D2 — `ValidateLoomDiscussion`. Its callers are `NewDiscussionGate` (the Discussion-Write and Discussion-Burler-Fix rows, `contracts/recipes/loom-recipe.yaml:110,200`) and `validate-discussion` (`internal/loomcli/validate.go:72`). **Survives.** Both callers go through the one function.
- D1/D2 — `ValidateDarnDiscussion`. It returns early on a defect today (`internal/loomshed/gates.go:141`); D2 changes that and keeps the history check only on a clean record. Its callers are `NewDarnDiscussionGate` and `validate.go:70`. **Survives.**
- D1 — headings that `lym loom decision add` writes. `addedDecisionHeading` writes `Added after Discussion (&lt;by&gt;, &lt;date&gt;): &lt;title&gt;` (`internal/loomcli/decision.go:706`). D1's `### Added after Discussion` exemption has to match on that prefix. **Survives**, as long as the exemption is a prefix match.
- D2 — `lym loom decision add`. Its record check is a seam that restores the record on findings (`decision.go:79`); D2 leaves it out of the new checks. **Survives.**
- D4 — the Env index on the discussion gate. `wiring.go:309` sets `PlanIndex` for every recipe, and `wiring.go:296` sets `WorktreeRoot`. The discussion gate entry (`internal/shedrecipe/entries_gate.go:139-155`) passes neither today, so the change has to add both. **Survives.**
- D4 — `NewSlottedIndex`. A type load that no caller covers waits for a gate slot (`internal/planglyph/index.go:25-32`). The record doesn't say whether the new resolve method type-loads. **Survives**, but this is unverified.
- D5 — guard scan. `onlyKeepLinesAfter` admits only keep lines today (`internal/impactset/guards.go:194`); D5 widens it to seam lines. **Survives.**
- D5 — `cmd/testtiming/keep.go:21`. This is a third parser of the same directive stack. It already admits any `//lym:` line between a keep line and its func, so `//lym:seam` passes. **Survives.** D5 doesn't name this parser, but none of its behaviour needs to change.
- D5 — reflective helpers in tier2 files, such as `internal/fabricengine/mergestate_integration_test.go:36` (`assertNoZeroFields`). The guard scan refuses a guard in a tier2 or tier3 file. D5 doesn't carry that restriction over to seam markers. **Survives.**
- D9 — `ValidatePlan` hands `planContextFindings` only the pending cards (`gates.go:260-269`). **Survives.**
- D9 — `ValidatePlanLensReview` wraps `ValidatePlan`. **Survives.**
- D9 — `ValidateReworkPlan` hands `planContextFindings` every card, `plan.Cards` (`gates.go:330`). **Breaks** D9's claim that done cards are skipped (finding 1).
- D9 — `--require-approved` calls `PlanIndex.Validate` with no `PlanContext` (`validate.go:283`). **Survives.**
- D3 — reading the git-tracked root entries from `internal/loomshed`. Nothing in loomshed reads git today, and `PATTERN-fabric-git` and `PATTERN-gitrepo-client-boundary` own git reads. **Breaks**: the record names no source for the tracked list (finding 2).

**Findings**

1. MEDIUM · design · D9, "They skip the cards webster's run record holds done, as the other tree-dependent checks do."
   - **Issue:** The `--rework` path hands D6–D8 the whole plan, done cards included, so they report findings against frozen done cards that `done-card-edited` stops the writer from fixing.
   - **Evidence:** `ValidateReworkPlan` calls `planContextFindings(plan, plan.Cards, …)` at `internal/loomshed/gates.go:330`. The rework plan keeps the earlier cards: `first_card` is one past the highest committed card (`gates.go:318-322`). `ValidatePlan` filters to pending cards only at `gates.go:260-269`.
   - **Fix direction:** D9 states how the rework path scopes D6–D8 to the cards from `told` onward. Otherwise the claim narrows to the gate and the no-flag mode.

2. MEDIUM · design · D3, "its first segment is an entry git tracks at the worktree root".
   - **Issue:** The check depends on git's tracked root entries, which keeps `.scratch/` and other untracked roots out of the test. The record doesn't say what supplies that list to `internal/loomshed`, and loomshed runs no git today. Running git there would go against `PATTERN-fabric-git` (git on warp or weft only via `internal/fabricengine`) and `PATTERN-gitrepo-client-boundary`.
   - **Evidence:** D3's text, and D4's "`internal/loomshed` tests them against the told worktree root and index". `ValidateLoomDiscussion` (`gates.go:123`) and the discussion gate entry (`entries_gate.go:155`) take only paths.
   - **Fix direction:** Name the source of the tracked-root list, for example a value or seam filled through fabricengine on Env and the verb's wiring. Or define the root test without git.

Tool calls:
1. grep for `ValidateLoomDiscussion`, `ValidateDarnDiscussion` and `testtiming:keep` callers: the manifest had no list of callers beyond `gates.go`.
2. `cmd/testtiming/keep.go` and `discussionparser` callers: the keep-line parser and the record check behind `decision add` weren't in the manifest.
3. `decision.go` validate sites, `discussionwrite.go` and the recipe gate rows: to find which rows run the discussion gate.
4. `recipe.yaml` rows and `loomcli` files that use `discussionparser`: to confirm which rows carry the discussion gate.
5. `NewSlottedIndex` and the `decision.go` function list: to see whether the index slots its type load, and what decision add checks.
6. `addedDecisionHeading` and the `validate.go` mode dispatch: the heading text and the mode-to-function mapping weren't in the manifest.
