# Card 19 — join-parts-verb

**Create:**
- `plan:internal/loomcli#joinPartsDeps` -> `type joinPartsDeps struct`
- `plan:internal/loomcli#joinPartsVerb` -> `func joinPartsVerb(out io.Writer, d joinPartsDeps) int`
- `plan:internal/loomcli#loomCLI.joinPartsCmd` -> `func (c *loomCLI) joinPartsCmd() *cobra.Command`
- `plan:internal/loomcli#TestJoinPartsVerb` -> `func TestJoinPartsVerb(t *testing.T)`

**Edit:**
- `plan:internal/loomcli#verbRowEngines`
- `internal/loomcli#refuseVerbOffRecipe`
- `internal/loomcli#Command`
- `internal/loomcli#TestArmAt_RefusesOffLoomRecipes`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/loomshed#JoinParts`
- `plan:internal/loomengine#PartsPlanRole`
- `internal/seatengine#SeatRole`
- `internal/loomcli#circlingStatusReader`
- `internal/loomcli#samplerStrandReader`

**Intent:** Add `lym loom join-parts` (D8 and its added decision), the chair's one call that joins the parts into the plan directory.

`verbRowEngines` maps `join-parts` to `PartsPlanWrite`, so `refuseVerbOffRecipe` refuses it on a recipe with no `Parts-Plan-Write` row with "loom: join-parts does not apply to this run, whose %q recipe has no Parts-Plan-Write row and so no parts to join; way forward: none is needed, and a run with a plan edits it with \"lym loom insert-card\" and \"lym loom remove-card\"".
`TestArmAt_RefusesOffLoomRecipes` gains that row.

`joinPartsDeps` holds `row` (the name of the recipe's `PartsPlanWrite` row, from `loomrecipe.EngineRow`), `readStatus` (the run's status, as `circlingStatusReader` reads it), `caller` (the calling session's `LYM_STRAND_NAME`, read with `os.Getenv(agentname.StrandNameEnv)` in the cobra shell), `chair` (the chair's strand name, formed with `agentname.Format` from the hub shortname and the task slug the seat geometry of `wire` uses and `seatengine.SeatRole(loomengine.PartsPlanRole, "")`, the name the seat engine gives the chair) and `join` (`loomshed.JoinParts` over the anchor, the worktree root, the plan index and the plan context `validate-plan` builds).
`joinPartsVerb` checks in this order and emits one envelope, each refusal naming its way forward:
1. The status is absent, or its current producer is not the row, or its state is not `running`, read from the status's state and current producer and never a row-name literal:
   - at the row in any halted state (blocked, failed, paused or awaiting), a run moved onto it by goto included: "the join runs only while Parts-Plan-Write is the running row; way forward: \"lym loom resume\", after which the row's chair runs the join: a live chair is resumed and joins itself, and otherwise the fresh spawn archives every part, rotates the plan directory and has the new chair join";
   - at another row, or with no status, which means no run has started here: "the join is the chair's alone, and a running Parts-Plan-Write row's chair runs it; past Parts-Plan-Write the joined plan is edited in place, for example with \"lym loom insert-card\"".
2. The caller is not the chair's strand name: the same "the join is the chair's alone" refusal, naming the caller, which covers a part writer, the operator's shell, orch, a tackler and an arbiter.
3. `join`'s refusal (a missing parts file or part overview, a failing part, an unmet export, a handle or path two parts create, a `language:` or `root:` mismatch) is passed through with its part and way forward.
Otherwise it prints `ok` with `plan_dir` and the joined cards, one `{part, from, to, file}` each.
`joinPartsCmd` runs on the full wiring, which tells it the reed geometry, takes no arguments, and is registered by `Command`; its help states the bound below.

Bound: the verb overwrites only the plan directory, and only from the chair of a running `Parts-Plan-Write` row, so a halted or later run, a part writer, a Plan-Burler-Fix fixer and Webster cannot reach it; within the row a rerun discards whatever the chair added after the last join, which the chair's stencil keeps it from doing.

`TestJoinPartsVerb` (tier0, over fake deps, as the renumber verbs' tests fake theirs) covers: the chair of a running row joining, with the envelope listing the cards; a blocked run at the row and a run at another row refused with their ways forward; a running row with a caller that is a part writer's strand refused; an absent status refused; and a `join` refusal passed through.

The loomcli doc and `docs/overview.md`'s loom verb list gain the verb and its bound.

**ImpactSummary:** One new chair-only verb; the refusal table gains its row, and no existing verb changes.
