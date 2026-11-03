```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I found no findings. Per change, the set I found and whether the diff covers it:

1. **Baseline stencil, Step 1 invariant rule**
   - Consumers: the renderer `BaselineTable` and `BaselineInputs` (`internal/driftengine/seats.go:116`), and the marker list in `contracts/stencils/markers_test.go:66`.
   - Result: covered. The diff adds no marker and drops none, so the render and the marker test are unaffected.
   - Docs: the `DriftTemplateBaseline` comment (`contracts/stencils/stencils.go:61`), `internal/driftengine/doc.go:61`/`:76`/`:91` and `template.yaml:6` describe the baseline agent as writing the "drift it expects" or "expectation". That still holds.

2. **Baseline stencil, Step 2 `full` bullet**
   - Neighbouring text: the generated no-agent report (`internal/driftengine/baseline.go:168-171`) says it "expects no particular drift per test". That is consistent with the new wording.
   - Pre-authorization wording: the repo-wide grep for "accepts the intended drift", "accepts each test" and "with a reason" finds no remaining baseline-side wording.
   - Recipe comments: `contracts/recipes/darn-recipe.yaml:119` and `tapestry-recipe.yaml:334` and `doc.go:115` say Drift-Verify's agent "accepts the intended drift and fixes the rest". That is the verify side, which still holds.

3. **Baseline stencil, Step 4 report rule**
   - The `drift scope` verb and `RecordScope` are untouched, and no Go code or test reads the report's prose.
   - `baseline_test.go` seeds "expected drift" as opaque bytes and asserts nothing on the stencil text.
   - `checkRecordedScope` (`baseline_test.go:170-182`) asserts only on generated-report fragments, and nothing in it covers the stencil prose.

4. **Verify stencil, Step 2 added rule**
   - Renderer: the `judging_rules` text (`seats.go:57-62`) is `autonomousVerifyJudging`, which tells the agent to accept with `--reason <text>`, and `operatorVerifyJudging`, which presents "the baseline report's expectation". Both agree with "expectation, never permission".
   - Neighbouring lines: the unchanged "Drift the task intends is acceptable" line and the "merged from the parent … acceptable too" line read consistently with the new rule, so I raise no finding. A pure-refactoring task's parent-merge drift stays acceptable, and the "too" still attaches to line 30.
   - Marker list: unchanged, and `markers_test.go:67` still matches.

5. **Repository rules and documents**
   - `PATTERN.md` has no entry that governs a stencil's prose, apart from `PATTERN-stencil-ownership`. The diff keeps that rule, since the stencils are still read at call time and nothing is embedded.
   - `PATTERN-parent-directive` still holds: both stencils keep `{{.parent_directive}}` and `{{.edit_directive}}`, and the diff touches no skills or color.
   - `CLAUDE.md`, `AGENTS.override.md`, `docs/skills.md` and `docs/code-comment-conventions.md` name no doc or spec to update for a drift-stencil change.
   - The diff declares no PATTERN changes, so there is nothing to land.
   - The commit message carries no `Pattern-Departure:` trailer, and the diff needs none, since it breaks no PATTERN entry.
   - The seeded Bolt copy under `_bolt/_lym/stencils/drift/` is a runtime seed outside this diff's range, so I raise no finding on it.

Tool calls:
1. `grep` over specs, contracts, pattern, `PATTERN.md`, docs and internal for baseline-report and pre-authorization wording. My context had only the earlier narrower grep.
2. `Read` of `internal/driftengine/seats.go` lines 45-75, for the actual `judging_rules` text my context lacked.
3. `grep` of the rule documents for stencil or drift requirements, plus an `ls` of the seeded Bolt stencil directory, to check for a mandated doc update.
