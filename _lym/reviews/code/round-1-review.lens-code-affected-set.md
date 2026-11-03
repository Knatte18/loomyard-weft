```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens: what the diff's changes force outside the lines they touch.

**Per change, the set found:**

- **Tapestry's Discussion rows move into the `Motif` block.**
  - Covered: `loomrecipe` names, `Rows`, `RecipeEngines` and `embeddedRecipes`; `BouncerRunSubdir`, which still finds `Discussion-Bouncer` through tapestry's expanded rows; `reportengine/timingspans.go`; the `loomcli` Discussion perch table and decision-add guard (`wiring.go:1085`, `decision.go:727`), both keyed on row names that did not change; `interruptpolicy.go`; and the tapestry header comments.
  - Covered: `docs/overview.md`.
  - Not covered: `docs/research/weaving-glossary.md` (finding 1).
- **New `undiscussed` gate and `Env.SeedDiscussedRecord`.**
  - Covered: `shedrecipe/doc.go`, `loomcli/doc.go`, `envkit.FullEnv`, and `battencli/wire_seams_test.go:73`.
  - The only non-test Env wirer is `loomcli/wiring.go:880`, so no other builder of tapestry loses the seam.
  - The `pattern/PATTERN-shed-recipe-registry.md` background was not updated (finding 2).
- **`motif` becomes a runnable recipe.**
  - Covered: `shedrun.RecipeNames`, `battencli/wire.go:532` and `:538` (Seed-Child and Run-Shed), `boardcli/cli.go:507`, `boardengine/task.go:319`, `shedcli/table.go`, `orch-role.md:43`, `plugins/ly/skills/board/SKILL.md:52`, and `docs/overview.md` at lines 262, 267, 537, 684 and 792.
  - Not covered: batten's handling of a child that finishes with no Finalize (finding 3).
- **Record-only loom contract and `Expand` ignoring a block recipe's top-level keys.** Covered by `shedbuild/doc.go`, `loomcontract.go` and the updated `expand_test.go` and `build_test.go`. The stale `shed-recipe-spec.md` text was updated in the same diff.
- **Finished-run draft status.** Covered by `loomcli/doc.go`, `wiring.go` and `arbiterprompt.go`. The batten and dobby consumers of that end state are unverified (finding 3).
- **arras removed.**
  - Covered: `recipes.go`, `names.go`, `seed.go`, `reportengine/runs.go` and `timingspans.go`, `tokencount`, and the specs and stencils.
  - The tree-wide grep leaves only `internal/shedrun/seed_test.go:174` and `docs/research/weaving-glossary.md:127` (finding 4).
- **Commit messages and PATTERN.**
  - None of the three commits carries a `Pattern-Departure:` trailer.
  - The directive declares no PATTERN changes, so there is nothing to land.

**Findings:**

1. **LOW · consistency · `docs/research/weaving-glossary.md:24`, `:37`, `:145`.**
   - These lines still say the recipe names are "tapestry, darn, sampler, pawl, batten" and that shed's recipes are the same list.
   - Line 145 still calls `motif` "reserved as a recipe name … which lets a shed block run a whole sub-recipe".
   - `motif` is now a runnable task recipe, in `RecipeNames` and the arming table, so these lines are wrong.
   - The same diff edited the glossary's arras row (`:127`) and the `cartoon` line (`:28`).

2. **LOW · design · `internal/shedrecipe/entries_block.go:46-60`, `recipe.go:257-260`.**
   - The `undiscussed` gate performs a side action: it writes the record, writes the support log, and commits.
   - It does so through a new single-purpose `Env` field.
   - `pattern/PATTERN-shed-recipe-registry.md` says "A row never gets its own Env field" and "A side-action row … is a `Command` row".
   - The summary admits judging this "no departure", but no `Pattern-Departure:` trailer records it and the pattern background is unchanged.
   - The answer key licenses the gate reading the record, so I rate this LOW: the process gap is the finding, not the behavior.

3. **LOW · design · `internal/battenshed`, `internal/battencli`; evidence: no Finalize-less handling found.**
   - A `motif` child ends with no Finalize and no landing commit.
   - My search of battenshed, battencli, dobbyengine and landingshed found no handling or test for Land, Worktree-Teardown, or the `notify_on_land` notice on such a child.
   - Their Finalize logic only covers the "Finalize past merge" cases (`innerrun_owner.go:176`).
   - `_lym/landing/summary.md` lists this as "To check by hand", so it is uncovered.
   - The same summary lists the board-entry checks the decision record asks for ("draft", run lock, dobby restart) as unverified end to end.

4. **NIT · scope · `internal/shedrun/seed_test.go:174` and `docs/research/weaving-glossary.md:127`.**
   - The test row `{"former arras name", "arras", true}` and the glossary row both still name arras.
   - The test row is a negative pin and the glossary row records that the name is free, so I would accept both.
   - The goal says to remove every place that names it, so I list them in case the handler reads that strictly.

**Tool calls:**
1. Grep for `RecipeSampler` and `RecipeDarn` in non-test Go: to find hard-coded recipe lists that might omit motif (my context had only the diff).
2. Grep for task-recipe lists in docs, stencils and the board packages: to find docs that name the recipe set.
3. Grep for the `Discussion-Write` and `Discussion-Bouncer` row names in non-test Go: to find readers of those names.
4. Grep for `ApproveDiscussion` in non-test Go, plus a grep for Finalize in battenshed, battencli, dobbyengine and landingshed: to find other Env wirers and Finalize-dependent end-state handling.
5. Read of `recipekeys.go` and `bouncerrows.go`: the per-recipe tables were not in my context.
