```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens. For each card I checked three things: reflective seam tests and test tier tags, doc text the card makes false that no card lists, and whether the card can pass its own gate given the cards before it.

**Per card**

- **Card 1:** Seams: no reflective test covers `SessionUsage`; held. Doc: `Result.Usage` (`run.go:295`), `UsageReader`, `shuttleengine/doc.go:287` and `claudeengine/doc.go:95` are all listed; held. Order: held.
- **Card 2:** Doc: `codexengine/doc.go` is listed; held. Tiers and order: held.
- **Card 3:** Seams: no reflective test covers `Runner` or `RunState` (the `reflect.DeepEqual` uses in `run_test.go` are value comparisons); held. Doc: `shuttleengine/doc.go` is listed; held. Order: held.
- **Card 4:** Doc: `PATTERN.md`, the three background files and the three `doc.go` files are listed; held. Order: held. Headroom is the gate's check.
- **Card 5:** No shared struct gains a field; held. Doc: `loomcli/doc.go` and `gatecli/doc.go` are Uses only, and the card only adds a hook, so held.
- **Card 6:** Seams: `landingshed.Deps.RecordLandedSize` is covered by `TestLandingDeps_EveryFieldPopulated` and `TestWire_EverySeamFilled`, both listed; held. Doc: held.
- **Card 7:** Seams: `configreg` tests are listed; held.
- **Card 8:** Held.
- **Card 9:** Seams: the template and classification tests are listed; held.
- **Card 10:** Held. `TestRegistry_RelPathPlacesBothUnderLoomFamily` (`contracts/specs/specs_test.go:53`) uses an explicit table, so it does not break.
- **Card 11:** Held.
- **Card 12:** Tiers: the new package's `testmain_test.go` is planned, and tiers are set per test through `tiergate.Require`; held.
- **Card 13:** Doc: **broken (LOW)**, see finding F3.
- **Cards 14–19, 20, 22, 24, 25, 26:** Held. Each new seam's reflective test (envkit, `TestWire_EverySeamFilled`) is listed where it applies. No reflective test covers battencli's `startGuard`.
- **Card 21:** Invariant tests: **broken (MEDIUM)**, see finding F1.
- **Card 23:** Doc and prompt text: **broken (MEDIUM, twice)**, see findings F2 and F4. Contract: `checkLoomContract`'s record-only form (`internal/shedbuild/loomcontract.go:58-62`) accepts three rows with no build or Finalize row; held.
- **Card 27:** Prosa only; held.

**Findings**

- **F1 — MEDIUM, consistency, card 21** (`contracts/stencils/raddle/raddle-template-agent.md` and `roleOpeningStencils`).
  - Card 21 maps role `raddle` in `roleOpeningStencils`, but its stencil content list never names the parent-directive or edit-directive markers.
  - `TestStencils_OpeningStencilsRenderParentDirective` and `TestStencils_OpeningStencilsRenderEditDirective` fail for any opening stencil that lacks either marker.
  - Evidence: `contracts/stencils/parentdirective_test.go:1-4,59,114-131`; `internal/seatengine/table.go:49` (seatengine fills `parent_directive`); `contracts/stencils/loom/loom-template-discussion.md` carries `parent_directive`.
  - Fix: the card's Intent names `{{.parent_directive}}` and `{{.edit_directive}}` in the stencil.
- **F2 — MEDIUM, consistency, card 23** (`internal/loomcli/doc.go:25`).
  - The package doc says "A finished run of a recipe with no Finalize row, a motif run, makes its entry a Sketch".
  - Card 23 makes `finishesAsSketch` false for a recipe with a `RaddleFile` row, which makes that sentence false. Card 23 lists `internal/loomcli/doc.go` only under Uses, and no other card edits it.
  - Evidence: `internal/loomcli/doc.go:25`; card 23, line 86 (Uses) and line 101 (Intent).
  - Fix: move `internal/loomcli/doc.go` to card 23's Extends or Edit, and reword the sentence.
- **F3 — LOW, consistency, card 13** (`internal/boardengine#StatusDraft`).
  - `StatusDraft`'s doc says "every new task takes it".
  - Card 13's `released` list on `applyDraftRule`, and card 16's filing through it, leave new tasks released, so the doc becomes false. Card 13 edits `applyDraftRule` but not the constant's comment.
  - Evidence: `internal/boardengine/draft.go:5`.
  - Fix: add `internal/boardengine#StatusDraft` to card 13's Edit.
- **F4 — MEDIUM, consistency, card 23** (`internal/loomcli#arbiterRecipeDescription`).
  - The arbiter prompt describes any recipe with no Finalize row as "a task's Discussion: its decision record is written, reviewed and stored on the board entry, and the run lands no code".
  - A raddle run has no Finalize row, so its arbiter would be told it runs a Discussion, when in fact it files tasks from a stored record. No card lists the function.
  - Evidence: `internal/loomcli/arbiterprompt.go:25,40-42`.
  - Fix: card 23 edits `internal/loomcli#arbiterRecipeDescription` to describe a recipe with a `RaddleFile` row by its rows.

**Tool calls**

1. Grepped the test lists of the stencil invariant tests (parent directive, role skills, Go values, markers, citations) and `specs_test.go`. The manifest did not cover contracts tests.
2. Read the spec family test, the markers table and the Go-values table, to see whether a new stencil or spec must join an exact table.
3. Grepped where `parent_directive` is filled, the reflective tests in battencli, shuttleengine and landingshed, and the doc text about record-only and no-Finalize recipes. None of this was in the manifest.
4. Read `checkLoomContract` and `recordOnly` in `internal/shedbuild/loomcontract.go`, `arbiterRecipeDescription`, and the reflect uses in `rundir_test.go` and `run_test.go`, to confirm card 23's contract claim and the doc and prompt text it makes false.
5. Grepped repository doc text on `UsageReader`, done-only usage and the draft rule, to find doc text cards 1, 13 and 16 make false outside the files they list.
