```yaml
lens: code-scope
outcome: reported
```

Code-scope lens: each change traced to the goal points under the board entry's `## Goal`. The record has no `Added after Discussion` entries.

**Per change**
- `boardengine/store.go` (`checkOfferedRecipe`, called from `validateWrite`) refuses arras with a hint naming motif. It traces to G1c. The forced edits that go with it: the `store.go` file header, the `boardengine/doc.go` paragraph, `TestUpsertRefusesTheArrasRecipe`, and the store-level arras set in `raddlefile_test.go` `startArrasRun`.
- `board-policy/SKILL.md` now proposes motif in place of arras, and the separate motif bullet is folded into it. Traces to G1a.
- Arras is dropped from the recipe lists in `board/SKILL.md`, `orch-role.md:46-48` and `boardcli/cli.go:581` (upsert help). Traces to G1b.
- `orch-role.md` drops the line on operator decisions for a running arras entry. Forced by G1 (the orch role no longer names arras).
- `orch-role.md:46` adds `raddle` to the list of inner runs. It traces to G1b and the Aim (motif then raddle is the route): the list changes there anyway, and the new motif sentence names raddle.
- `RunView.Motif` (`passes.go`), its read in `battencli/dobbyreads.go`, the Sketch line in `rememberRuns` and `sketchNoticeLine` (`landnotice.go`), and the `dobbyengine/doc.go` and `landnotice.go` headers. Traces to G2a. The forced test edits are the `TestLandNotices` rows and the renamed `TestDobbyRuns_…MotifRecipe` with its motif assertion.
- `orch-role.md:96` (notice list) and `orch-role.md:361-365` (After landing: put the Sketch to the operator). Traces to G2a and G2b.
- `role-autonomy.md` section "A Sketch a motif run left", steps 1–2. Traces to G2c.
- `role-autonomy.md`: the intro line, the split-ready notice and the Slack line. Traces to G2d (split approval stays the operator's) and the Aim.
- `raddle-template-agent.md`: Step 1 superseded decisions; Step 3 size, tapestry, recipe/profile, order/deps, lands whole, map clauses, no invented parts, size estimates; Step 4 wording and filing level. Traces to G3, and each item the entry lists is present.
- `loom-rubric-raddle-review.md`: the new judgement bullets, the replaced recipe/profile bullet and the extended `design` class list. Traces to G3 (the split perch's rubric). The class-list extension is forced, since a new judgement needs a class.

**Per decision**
- G1a: built by the board-policy edit.
- G1b: built by the board skill, orch role and `lym board` help edits.
- G1c: built by `checkOfferedRecipe` and its test.
- G1 "nothing visible names it … recipe listings": only partly built (finding 1).
- G2a: built by the dobby Sketch line and `RunView.Motif`.
- G2b: built by orch-role After landing.
- G2c: built by role-autonomy, read differently from the entry's wording (finding 2).
- G2d: built by role-autonomy's split-ready passage.
- G3: built by both stencil diffs.

**Findings**

1. **MEDIUM · scope.** Location: `internal/battencli/wire.go:616`.
   - Summary: G1 asks that no recipe listing name arras, but one error message still offers it, and the diff leaves it alone. The refusal for an unsupported child recipe tells the user "a Board task's type must be … `arras` … or empty", which lists arras as a recipe a Board task may take. `runprofile/template.yaml:2-4,26` is also untouched: its seeded `profiles.yaml` comments, shown in the `lym config profiles` editor, describe an arras block. The new board refusal means no write can now give an entry that recipe, so the listing also contradicts it.
   - Evidence: `internal/battencli/wire.go:616` and `internal/runprofile/template.yaml:2-4`. The board entry's Goal §1 reads: "nothing visible names it: … help text and recipe listings."

2. **LOW · decision.** Location: `contracts/orchrole/role-autonomy.md` (new section "A Sketch a motif run left", step 2).
   - Summary: the entry says the orch promotes the Sketch "once nothing it waits for is open". The diff has it promote at once, with `depends_on` naming the entries still open. The effect is the same: the promote releases the task, and dobby waits on `depends_on` (`boardcli/cli.go:192-196`). But it departs from the entry's literal wording. The change description raises it as the writer's own design call, and the record holds no entry for it.
   - Evidence: `_lym/landing/summary.md` "Design decisions", bullet 1, and the board entry's Goal §2, bullet 3.

No change traces to nothing: every hunk builds a goal point or is forced by one.

Tool calls:
1. grep of `internal/shedcli` for `Long:`, `Use:` and `names()`, to check whether `lym shed` help lists arras. The manifest held only `table.go`'s entries.
2. `sed` of `internal/shedcli/seed.go:108-140`, to read the `lym shed seed` help text for a recipe listing. It names no arras.
