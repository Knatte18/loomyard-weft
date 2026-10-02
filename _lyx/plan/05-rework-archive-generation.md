# Card 5 — rework-archive-generation

**Edit:**
- `internal/loomshed/prrework.go`
- `internal/loomshed#ReworkTold`
- `internal/loomshed/prrework_test.go`
- `internal/loomengine/rework.go`
- `internal/loomengine/rework_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_prrework.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/reject.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/shedbuild/fixture_test.go`

**Delete:**
- `internal/planparser/appendonly.go`
- `internal/planparser/appendonly_test.go`

**Prosa:**
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/specs/refusal-spec.md`
- `CONSTRAINTS.md`
- `docs/overview.md`
- `README.md`

**Uses:**
- `internal/planparser#ReviewExempt`
- `internal/websterengine#ArchiveRunRecord`

**Intent:**
Make a PR-Rework round start a new plan generation instead of appending cards (D1, D2, D4, D8, D14), retiring the append-only machinery.

Archive layout, declared once as constants in `prrework.go`: round `N`'s directory holds `findings.md`, `record.json`, `coverage.md` and `prior-generation/`, which holds `plan/` (every entry of `_lyx/plan/`: cards, `00-overview.md`, `amendments.md`, any `archive-*/` rotation), `webster/` (webster's run record) and `reviews/<run_subdir>/` (the Plan-Review and Webster-Review run directories).

`PRReworkDeps` loses `Rebaseline` and gains `ArchiveWebster func(dest string) error`, `ReviewsDir` (the absolute reviews root) and `ReviewRunSubdirs` (the run subdirectories to archive; the wiring tells `plan` and `webster`, the recipe's `run_subdir` values for the two segments).
`ReworkTold` gains `PriorPlanDir`, the archived plan directory the session reads.

`prRework.Call`, after the existing pending-rejection and committed-round checks:

1. Picks the round per D14: a working-tree round whose `record.json` records the pending rejection's identity means the archive is done and the session may have begun, so nothing is moved again; otherwise the highest round directory with no `record.json` is this rejection's interrupted archive and is resumed; otherwise a new round, highest plus one.
2. When the archive is not done: reads `first_card` with `NextReworkCardNumber` (the plan committed at HEAD is still the retired generation), writes `findings.md`, moves every plan entry, calls `ArchiveWebster` with the round's `prior-generation/webster`, moves each review run directory, and writes `record.json` (`pr_number`, `head_sha`, `rejected_at`, `first_card`) last, as the archive's completion marker.
   Each move skips an entry already moved; an entry present at both origin and destination is a returned error naming both and the way forward (remove whichever copy is stale, then re-step).
3. Builds the session from the factory, telling it the recorded `first_card` and `PriorPlanDir`.
4. After the session returns Done (its gate has passed), parses the live plan, classifies it with `ReviewExempt`, copies `coverage.md`, rewrites `record.json` with a `class` field added (`exempt` or `required`), commits, then removes the pending rejection and returns Done.

`committedRejections` counts only records at HEAD that carry a class, per the overview's `committed-round-carries-a-class`: card 16 keeps the status commit from landing any of the round while its rejection is pending, but a classless record can still reach HEAD outside that window, and counting it would skip the session.
`appendOnlyViolation`, `stuckReasonParse`, the `Rebaseline` call in `finish`, and `planparser.CheckAppendOnly` with its test are deleted.
Bound on removing the append-only guard: the built cards are no longer in `_lyx/plan/` when the session runs, so it cannot rewrite them there; the residual is a session that edits the archive it reads, which the round commit shows as a modified rename in the PR.

`loomengine.ReworkSpec` takes `firstCard int, priorPlanDir string`; the stencil's `next_card_number` marker becomes `first_card` and a required `prior_plan_dir` marker is added.
The rework stencil is rewritten per D4: write a complete plan into the empty `{{.plan_dir}}` (a fresh `00-overview.md` with `approved: false`, `first_card: {{.first_card}}`, a Card Index listing only the new cards numbered from `{{.first_card}}`, plus the new card files); read the archived overview and cards in `{{.prior_plan_dir}}` for context and carry forward whatever task framing, `## Shared Decisions`, `## Rename mechanic` and `## verify:` content still holds; never copy a retired card; never edit, move or delete anything under `{{.prior_plan_dir}}`; a file already in `{{.plan_dir}}` is a leftover of an interrupted attempt of this same round, which the session owns; `lyx loom validate-plan --rework` checks the whole new plan.

`wire()` fills `ArchiveWebster` with `websterengine.ArchiveRunRecord(websterGeom, dest)`, `ReviewsDir` with `loomengine.LoomReviewsDir`, and `Commit`'s pathspec with the plan directory, the rework directory, the reviews root and webster's durable directory, so the moved-from deletions and the archive land in one commit (card 16 keeps the status commit from landing part of it first on an interrupted round); an entry matching nothing in the index or the working tree is left out, since git refuses such a pathspec (see `statusCommitPathspec`).
The `CommitWebster` comment no longer cites the append-only check.
`prReworkEntry` requires the new seam and an absolute `ReviewsDir` in place of `Rebaseline`.

Tests in `prrework_test.go`, over temp directories with fake seams: a first round moves generation 0 whole (an `archive-*/` rotation and `amendments.md` included), webster is archived to the told destination, `record.json` carries identity, `first_card` and class, and `Commit` runs once; two consecutive rounds leave `round-1/prior-generation/plan` holding generation 0, `round-2/prior-generation/plan` generation 1 and `_lyx/plan/` generation 2 only; crash-resume converges to one round directory and one commit for a crash mid-archive (some entries moved, no `record.json`), after the archive and before the session (a file the session wrote into `_lyx/plan/` is never moved), after the session and before the commit, and after the commit and before the rejection is cleared; a classless `record.json` at HEAD does not count as a committed round; an all-`Prosa` generation records `exempt` and a mixed one `required`; a collision returns the named error.
`rework_test.go` asserts the prompt carries the told first card and prior plan directory.
The rework integration test drives two rounds through `wire()`'s real seams over a real fabric pair and asserts one weft commit per round carrying both the moved-from deletions and the archive, webster's directory emptied, and the live plan holding only the newest generation.

Docs, same commit: `CONSTRAINTS.md` gains the Plan Generation Invariant worded as D15 states it (`_lyx/plan/`'s top-level files hold exactly one generation; a retired generation lives only under the rework round that retired it, except where Plan-Write's own `archive-*/` rotation moves the live plan aside on a re-run; only PR-Rework archives a generation into a round; Webster's run record and the plan and webster review run directories are generation-scoped).
`docs/overview.md`'s rework paragraph, `README.md`'s rework line and `lyx loom reject`'s help text describe generations, not appended cards.
`refusal-spec.md`'s loom section gains rows for the archive collision and for a webster run holding its run lock during the archive (way forward: wait for it, then re-step), each with its reaching test here.

**ImpactSummary:** rewrites PR-Rework's Go-owned steps and its session's contract; reaches the PRRework registry entry, `wire()`'s rework seams, the rework stencil and spec factory, and test fixtures; deletes the append-only check, whose only caller was PR-Rework.
