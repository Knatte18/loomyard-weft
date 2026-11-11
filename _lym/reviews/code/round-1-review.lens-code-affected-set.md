```yaml
lens: code-affected-set
outcome: reported
```

Task: find what the diff's changes force outside the lines it touches (lens `code-affected-set`).

**Affected set, per change**

1. **Field `tierRun.ignoresTargets` removed.**
   - Set found: `tiers.go`, and `describe` in `tiers_test.go`.
   - Covered. Nothing else in the repo references the field.
2. **Run-whole notice moved from `runListed` (`run.go`) to `planTierRuns` (`tiers.go:146-148`).** It now prints at plan time, before any run, instead of when each run starts.
   - Set found: three `wantStderr` rows in `tiers_test.go` (lines 201, 258, 326). Covered, and they assert the new text.
   - Set found: the missing-build-dir assertion in `test_integration_test.go` (line 971 onward), which now reads the last stdout line. Covered.
   - Set found: the notice-asserting integration subtest deleted at the old line 1802. The notice stays covered by the three rows above. The through-the-process route is no longer pinned, which is within the test-economy rules.
   - Docs and stencils: a repo-wide grep for the notice text finds no stencil, spec or doc outside `gatecli` that quotes it. Covered.
3. **New refusal: explicit targets that no kept tier takes (`tiers.go:142-145`).**
   - Set found: `doc.go` lines 139-143 and 225-232 are updated. Covered.
   - Set found: callers that pass explicit targets to a tier with no placeholder. The remaining integration calls with `./pkg` (lines 894, 968, 2123, 2417) all include `--tier 0` or hit the `{args}` refusal first. Covered.
   - Set found: `PATTERN-refusal-way-forward`. The refusal names the way forward, and `tiers_test.go` pins it with an `exact` row. Covered.
   - The `planTierRuns` doc comment (`tiers.go:81-82`) is updated. Covered.
4. **New dotnet unreached-project lines (`run.go:222-226`, `dotnetscope.go`).**
   - Set found: `doc.go` 186-196 documents it. Covered.
   - Set found: `gateslot/template.yaml` and `gateslot/doc.go` describe `{targets}` placeholders but make no claim about run-whole behaviour. Nothing is left wrong.
5. **PATTERN and commit rules.** `pattern/PATTERN-test-speed.md` mentions only `{test_targets}` for go. No PATTERN entry is declared changed, and the commit messages carry no `Pattern-Departure:` trailer. Nothing to check.

**Findings**

None. Nothing outside the touched lines is left wrong by this diff.

**Tool calls**
1. A repo-wide grep for `ignoresTargets`, the notice text and "runs once, whole": to find every reader, asserter and doc of the changed behaviour, which the manifest did not cover.
2. A grep for files naming `{test_targets}` or the new line text: to look for specs, stencils or docs outside `gatecli` that describe the behaviour.
3. A grep for `targets}` in `gateslot/doc.go`, `template.yaml`, `impactset/doc.go`, `PATTERN-test-speed.md` and `tierrun.go`: to check that those mentions don't contradict the new behaviour.
4. A grep for `--tier 2` and `tier2` in the `gatecli` tests: to find other tests that pass explicit targets to a tier with no placeholder.
