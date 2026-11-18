```yaml
lens: coverage
outcome: reported
```

Coverage lens on the plan in `_lym/plan`: I checked each decision, acceptance criterion and new branch against the planned test rows.

**Findings**

1. **MEDIUM**, D1 / card 3: no row reaches the step that joins two warnings.
   - The `ensureRunsStrand` step 4 joins the runs sidecar warning and the re-add warning with `; `. No row fails both writes together, so that path is never reached.
   - Evidence: card 3's `blockRunsSidecar` row fails only the runs sidecar write and keeps the re-add succeeding. The `addUnlessErr` and `blockStatusSidecar` rows leave the runs sidecar write succeeding.
   - The record asks for the join: "a failed runs sidecar write's warning is joined with the re-add's own".
   - Fix: add a row that sets both `blockRunsSidecar` and `addUnlessErr` and asserts both warnings appear.

2. **MEDIUM**, D1 / card 3: no row reaches `selfRunUnfinished`'s unreadable-status branch.
   - Card 3 says "An absent, unreadable or done status is false", but no row writes garbage bytes as the self status.
   - Evidence: the `selfState` rows always write a well-formed status. The absent case is reached only by the existing row "a loom status strand holding the overview is replaced by the runs strand", which has no `selfState`.
   - Fix: add a row with a live `loom-status` strand and garbage at `shedrun.StatusFile(h.cli.location, shedrun.SelfRunID)`. It should expect calls `replace` with no `add-unless`, and no status sidecar.

3. **MEDIUM**, D2 / card 2: deleting the old test drops coverage of the placement scan when no strand matches.
   - The placement scan moves from `statusStrandAddSpec` into `ensureStatusStrandOn`. The deleted `internal/loomcli#TestStatusStrandAddSpec` had the row `OtherStrandsLeaveItTheOverview`: a non-runs strand keeps the overview placement.
   - Card 2's own rows have only "no strands" and "a live runs strand", so a non-empty scan that finds no runs strand is never reached.
   - Card 2 claims card 1's test carries the deleted cases, but card 1's builder takes a placement, not a strand list, so it cannot.
   - Evidence: `internal/loomcli/bootstrap_test.go:666-695` (the deleted test) against card 2's row list.
   - Fix: add a `TestEnsureStatusStrandOn` row holding only an orch strand (`tst:main:orch`, live) that expects the overview display.

4. **MEDIUM**, D3 / card 4: no row reaches the blank-line separators the new `frame` adds.
   - `frame` now places the self-run line between the dobby lines and the orch lines, with a blank line between each two sections present.
   - Evidence: `TestOverviewSource_DrawsTheSelfRunLine` writes no orch state, so its frames hold the self-run line alone. `TestOverviewSource_DrawsTheOrchLine` has no seed, so it never draws the self-run line.
   - Fix: add a row, or a frame in the new test, that writes an orch state for `beam-b` and asserts the exact lines: the self-run line, `""`, then the orch line.
   - The record's "renders the rest as before" for no-seed and done runs is only covered indirectly, by the orch-line test's prime having no seed.

5. **LOW**, card 2: `ensureStatusStrandOn` exposes several exits through its new seams, but no row reaches them.
   - The exits:
     - the `Status` error return;
     - the `executable` seam's error return;
     - the `currentBuild` error, which writes no sidecar and replaces a live strand;
     - the keep path's early return, where a live strand is current and nothing is called;
     - the `ReplaceStrand` failure, which only warns and returns nil.
   - Evidence: card 2's row list has five rows: add with no strands, add beside runs, `held`, replace of a dead strand, and `addErr`. None of them covers these exits.
   - The keep and replace decisions are pinned in `TestResolveStatusStrandAction`, but `ensureStatusStrandOn`'s handling of them is not.
   - Fix: add rows for a current live strand with a matching sidecar (no calls, nil), a replace error (nil, no sidecar), and a build-seam error (an add with no sidecar).

6. **LOW**, `PATTERN-no-hard-coded-choice` / cards 1, 3 and 4: every row seeds `shedrun.RecipePawl` and expects `pawl`.
   - A builder that hard-coded the `"pawl"` literal as `Window` would pass every row. The D2 rationale cites this pattern for reading the recipe from the seed.
   - Evidence: card 1's rows ("beside-runs with a pawl seed … `Window` `pawl`"), card 3's `wantStatusAdd` display `Window: "pawl"`, and card 4's rows built from `shedrun.RecipePawl`.
   - Fix: give card 1's table one beside-runs row seeded with another recipe, such as `shedrun.RecipeTapestry`, and expect that name as `Window`.

7. **LOW**, D1 / card 3: the absent-self-run criterion is met only by accident.
   - "With the self run done or absent … adds no status strand": the absent half rests on the existing row "a loom status strand holding the overview is replaced by the runs strand". That row is left without `selfState`, and card 3 never names it as carrying the criterion.
   - Evidence: card 3 lists the rows that gain `selfState` running, and this row is not among them.
   - Risk: an implementer who adds `selfState` to every existing row would silently drop the criterion.
   - Fix: card 3 should name that row as the absent-self-run case and keep it without `selfState`.

Further tool calls made: none.
