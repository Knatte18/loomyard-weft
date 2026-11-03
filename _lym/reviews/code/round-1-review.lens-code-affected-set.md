```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff forces outside the lines it touches.

**Per change, the set found**

- **`composeReviewPrompt` gained an `assistRules` argument.**
  - Callers: `review.go:108`, `assist.go:135`, and the tests in `template_test.go`, `prompt_test.go` and `assist_test.go`.
  - All are updated. Covered.
- **`newSeatRun` gained an `endMarker` argument.**
  - Callers: `engine.go:313`, `resume.go:121` and `engine_test.go:812`.
  - All are updated. Covered.
- **`CapAssistTimeout` and `loomSegmentSettings` signatures.**
  - Callers: `wiring.go:571`, `wiring.go:928` and the tests.
  - Covered.
- **`BurlerHalfDeps.Assist` and `SegmentSettings.Assist`.**
  - Constructors: `entries_burler.go:129` sets `Assist`. The fix row and any other `BurlerHalfDeps{}` literal leave it at the zero value, which is correct.
  - Covered.
- **The `burler-step-review` stencil gained the optional `assist_rules` marker.**
  - The only renderer is `composeReviewPrompt`, which passes the optional list. The commit `bfc719e3` fixed the tests.
  - Covered.
- **New stencils and engine blocks.**
  - They are registered in `stencils.go` `entries` and the `markers_test.go` rows.
  - `roleOpeningStencils` gains `burler-review-assist` and the assisted chair stencil, so the parent and edit directive tests reach them.
  - Covered.
- **The edit-directive restatement test exempts `path.Base(name) == "edit-directive"`.** This covers the new engine copies. Covered.
- **New config keys (`loom.yaml` and `tapestry.yaml`).**
  - Template, `ConfigSpec` categories, `OpenMaps`, `fanKeys`, `LoadConfig` validation and the `WithTapestry` copy are all updated.
  - `internal/configreg/configreg_test.go` is touched in the diff, so the keys are treated as covered there.
  - **Not covered:** `docs/overview.md` (lines 625, 628 and 672) describes loom's and tapestry's config keys, and the diff does not touch `docs`. That is finding F1.
- **`seatengine` assistant kind.**
  - `doc.go` is updated in the diff.
  - The `Result.Advisors` doc comment is updated, `work.go` outstanding is updated, and `resume.go` handles a lost assistant.
  - No other `SeatResult` consumer needs a change: `keepAssistReports` is the only new reader.
  - Covered.
- **`ReviewResult.Assists`.** Nothing else reads it. Covered.
- **PATTERN declared changes.**
  - The decision record declares `PATTERN-fs-links` (changes) and `PATTERN-engine-blocks` (adds).
  - Both entries already stand in `PATTERN.md` (lines 15 and 63) from earlier parts, and the diff does not touch `PATTERN.md` or `pattern/`.
  - Nothing is left to land. No finding.
- **Commit messages.** There are no `Pattern-Departure:` trailers, so there is nothing to judge.
- **The archive set.** `burlerreview.go` adds `AssistCopyPath` for seat `assist` to `paths`. That is the right set for `assist`, the only seat name. Covered.

**Findings**

- **F1 — LOW, class `scope`.**
  - Location: `docs/overview.md:625`, `:628` and `:672`.
  - Summary: the loom and tapestry config-key description in the overview does not mention the new assist keys (`discussion_assist_reviewer_model`, `code_assist_reviewer_model`, `plan_assist_reviewer_model`, the three `_fan` partners, `assist_reviewer_timeout_min`).
  - Fix: add the keys to those passages, as the `reviewer_fan` keys already are.
  - Evidence: the grep shows these lines list the sibling keys, and `git diff --stat` for `docs` returned nothing.
  - Caveat: the repository's own rule that docs land in the same diff was not verified, because I did not read any `CLAUDE.md` or docs rule. So this is LOW rather than MEDIUM.

No other affected-set gaps found.

**Tool calls**
1. `grep` for callers of the changed signatures, `PATTERN.md` entries, docs naming `code_reviewer_fan`, and `roleOpeningStencils`. My context lacked callers outside the diff.
2. `grep` of `docs/overview.md`, `git diff --stat` of `docs`, `PATTERN.md` and `pattern`, and other `BurlerHalfDeps` constructors. I needed to confirm that docs and `PATTERN.md` were untouched and that no other constructor sets `Assist`.
