```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each thing the diff changes, find what reads or asserts it and report whatever the diff leaves wrong.

**Per change, the affected set:**

- **New `Profile.Diff` field (unexported `diffPieces` written in `Engine.Review`).**
  - `composeReviewPrompt` and `diffPiecesBlock` read it: covered.
  - `composeAssist`/`assistProfile` copies `p` after `diffPieces` is set, so the assistant's explore step names the same pieces: covered.
  - `probeTable`/`SeatTables` compose no prompt, so they are not affected: covered.
  - `review.go` `Review` doc and `doc.go` were updated: covered.
- **New `diff/` subdirectory inside the `review-*` instruction directory.**
  - Only `instructionDir` creates it, and nothing lists or sweeps that directory's files: covered.
  - It sits under `.lym/burler` through `lymdirs.DotLymDirName`, as `PATTERN-lym-files` requires: covered.
- **New explore marker `diff_pieces`.**
  - Explore stencil header and marker docs updated: covered.
  - `prompt_test.go`'s optional list updated: covered.
  - `template_test.go` `instruction1MarkerValues` and its doc, plus the `TestTemplate_OptionalDirectives` doc and row, updated: covered.
  - `TestTemplate_FillsWithAllMarkers` (doc at `internal/burlerengine/template_test.go:89-92`): not updated. See F3.
- **New `burler-diff-pieces` stencil.**
  - Registered in `stencils.go`'s embed and entries, and pinned by `markers_test.go`: covered.
  - Its wording against `loom-rubric-code-review.md`'s range rules: not covered. See F1 and F2.
- **New `stage_diff` row key.**
  - `configRejectUnknown` list, `shedrecipe/doc.go` and the `burlerReviewEntry` doc: covered.
  - The three recipes, plus the `loomrecipe/testdata/tapestry-rows-before-motif.yaml` fixture: covered.
  - No other recipe fixture carries `code_fixer_fan`.
  - Motif and arras review rows have no code subject and do not set the key: correct.
- **`BurlerHalfDeps.Diff`.**
  - Constructor, test spec and `shedadapters/doc.go`: covered.
- **PATTERN.**
  - No declared changes and no `Pattern-Departure:` trailers.
  - `stage_diff` is a row config key, not an `Env` field, so `PATTERN-shed-recipe-registry` holds.
- **Commit messages.** They match the diff.

**Findings:**

- **F1, MEDIUM, design.** Location: `contracts/stencils/burler/burler-diff-pieces.md:55-59`.
  - The stencil says "The diff under review is already written out below" and tells the reviewer to "Read these pieces in place of the review range's `git diff`".
  - The pieces are the full task patch against the merge base (`internal/loomcli/subjects.go:161-180`).
  - For a merge-parent task the rubric narrows the range to the merge itself (`contracts/stencils/loom/loom-rubric-code-review.md:38-40`). The pieces then hold more than the range and are mislabelled as "the diff under review".
  - For the `none` base, the rubric says to review nothing, yet the stencil still offers pieces as the subject.
  - The stencil never tells the reviewer to defer to the rubric's range when the two differ. The summary claims both cases "still go through its own rules", and the stencil text does not bear that out.
- **F2, LOW, consistency.** Location: `contracts/stencils/burler/burler-diff-pieces.md:57`.
  - The pieces replace the rubric's committed `git diff <base>..HEAD`.
  - The patch, however, is the working tree against the base plus untracked files (`internal/fabricengine/taskpatch.go:83-99`).
  - Uncommitted or untracked content would show up in the pieces but not in the committed-diff subject the rubric defines ("The subject under review is the committed diff").
- **F3, NIT, consistency.** Location: `internal/burlerengine/template_test.go:89-92`.
  - The diff adds `diff_pieces` to this test's optional list (around line 139).
  - Its doc comment still says the explore set includes only "the optional pattern_directive and focus_directive" and that only those two are excluded from the deletion sweep.
  - The neighbouring `instruction1MarkerValues` doc was updated, so this one is inconsistent with it.

**Tool calls:**
1. Grepped for files naming `fork_reading`, recipe fixtures carrying `code_fixer_fan`, and `burler-fork-reading` users. I needed the docs, fixtures and registries that mirror the new marker, row key and stencil, and my context lacked them.
2. Grepped for `reviewer_fan_setting` in markdown specs and for `burlerDirName`/`probeDirOf` users. I needed to know whether any spec lists `BurlerReview` keys and whether anything sweeps the instruction directory, which would now hold `diff/`.
3. Read `template_test.go:86-93`, `doc.go:455-465` and the explore stencil's marker-ordinal lines. I needed the exact doc text of the test whose optional list the diff changed, which my context held only as diff hunks.
