```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change the diff makes, the set that reads or asserts it, and whether the diff covers it.**

- **`{targets}` test replaced by `gateslot.HoldsTargets`:** a search of non-test code found no other site that tests `"{targets}"` by `strings.Contains`. The sites in `green.go`, `tiers.go` and `gateslot/tiers.go` are all switched. Covered.
- **`renderTierCommand` signature:** the only non-test caller is `tierrun.go:286`, and the tests in `tiers_unix_test.go` are updated. Covered.
- **`expandPackages` moved from a method to a function:** all four call sites in `green.go` are updated. Covered.
- **New `{test_targets}` placeholder:**
  - It is documented in `gateslot/tiers.go`, `template.yaml`, `gatecli/doc.go`, `impactset/doc.go` and `PATTERN-test-speed.md`.
  - The `all_targets` validation and its test rows are updated.
  - The hub's real tier commands are not in this repo (the template declares no tiers), so the diff cannot show whether every tier's command now uses `{test_targets}`. This is outside my lens and left to the handler.
  - Covered, except for the stale items below.
- **PATTERN entries:** `{test_targets}` is additive, so `PATTERN-config-layers` needs no reinterpretation. The commit message carries no `Pattern-Departure:` trailer and none is needed. No `PATTERN.md` change is declared in the rubric. Covered.

**Findings**

1. **LOW, consistency:** `internal/gateslot/doc.go:138` still says `all_targets` is required "when its command holds `{targets}`". The check now also fires for `{test_targets}`, as the new text at `gateslot/doc.go:110-112` and the refusal message at `gateslot/tiers.go:187` say. Evidence: `gateslot/doc.go:138`.
2. **LOW, consistency:** The guard-step refusal still names only `{targets}`, though `HoldsTargets` accepts `{test_targets}`.
   - Message: `internal/gatecli/tiers.go:165` says "which must hold {targets} and {args}".
   - Comments: `tiers.go:80` and `gatecli/doc.go:309`.
   - Fix: reword to "{targets} or {test_targets}".
3. **LOW, consistency:** Prose and a message still key whole-run behaviour on `{targets}` alone.
   - Comments: `tiers.go:192` and `gatecli/doc.go:132`, `:176`.
   - `ignoresTargets` doc at `tiers.go:53`.
   - Message: `internal/gatecli/run.go:486` prints "the command of … holds no {targets}, so it runs whole".
   - Behaviour already follows `HoldsTargets`, so a `{test_targets}`-only command is described wrongly.

**Tool calls (1)**
1. A `grep` for `{targets}`, `renderTierCommand(` and `.expandPackages(` across non-test Go, md and yaml files. My context lacked a repo-wide caller and doc search.
